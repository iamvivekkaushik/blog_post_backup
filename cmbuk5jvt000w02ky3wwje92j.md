---
title: "Internet Failover with Netplan and iptables on Ubuntu 24.04"
datePublished: Fri Jun 13 2025 08:40:52 GMT+0000 (Coordinated Universal Time)
cuid: cmbuk5jvt000w02ky3wwje92j
slug: internet-failover-with-netplan-and-iptables-on-ubuntu-2404
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1749801363161/4c1c2ca1-5042-43ec-bf2c-1455c2f747b7.png
tags: ubuntu, router, internet, networking, netplan, internet-failover

---

Lately, my internet has been throwing tantrums — not because the connection itself is bad, but because every time the power blinks, my router decides it's time for a little nap. Naturally, the obvious solution was to slap a power backup on the router (which I did — not becuase my plan didn’t work but because my secondary router is a 4G router and has the speed of a potato on dial-up).

But all this got me thinking: what if I could set up a proper network failover? You know, like having a backup parachute in case the first one gets stage fright. That would be pretty cool — automatic protection against actual internet outages, and I could keep streaming, working, or doom-scrolling without missing a beat.

Here is what I ended up with

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749801534798/888c4be3-8acd-479f-9ba5-7d1baef54db0.png align="center")

There was a lot of hacking involved to get to this point, let’s dive in.  

### The First Problem (May Not Be a Problem for You)

So the first issue I ran into — which you *might* not face — was that my portable Jio router decided to play hide-and-seek by changing its interface name and MAC address every time it rebooted. Super helpful, right? This made it impossible to apply any persistent configuration.

If your setup doesn’t suffer from this little identity crisis, you can probably skip this part. But if you do — welcome to the club — here’s how I fixed it using **udev rules**.

**Step 1: Identify your device**

First, you’ll need to find your device info:

```bash
udevadm info -a -p /sys/class/net/enxXXXXXXX
```

(Replace `enxXXXXXXX` with your actual interface name.)  
Look through the output and note down the important bits, like:

```bash
ATTRS{idVendor}=="0bda"
ATTRS{idProduct}=="8153"
ATTRS{serial}=="000001"
```

**Step 2: Create a udev rule**

Now that we’ve got what we need, let’s make the rule:

```
sudo nano /etc/udev/rules.d/70-persistent-net.rules
```

Add this line:

```bash
SUBSYSTEM=="net", ACTION=="add", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="8153", ATTRS{serial}=="000001", NAME="usbnet0", RUN+="/sbin/ip link set usbnet0 address 02:11:22:33:44:55"
```

* Replace `02:11:22:33:44:55` with whatever MAC address you want.
    
* `usbnet0` is a custom interface name that will now stay consistent (finally).usbnet0 is a custom interface name that will now always be used.
    

**Step 3: Reload rules or replug device**  
You can reload the rules like this:

```bash
sudo udevadm control --reload
sudo udevadm trigger
```

Or just take the lazy way like I did and unplug/replug the USB device. Done.

### The Second Part: Enter the Failover Magic

Now that we've successfully tamed our devices and given them proper names (so they finally know who they are), it’s time for the fun part — **creating the actual failover system.**

At this stage, both internet connections are plugged in and ready, but here’s the catch: Linux will only use one of them for traffic at any given time. This decision is made by something called the **routing table**. Think of it like Google Maps for your packets — it tells your data which road (interface) to take.

Now, in Linux, there’s a thing called the **default route** — basically the "main road" your traffic takes if there are no better instructions. You can have multiple default routes, and each can be assigned a weight (or "metric"). The lower the weight, the more Linux favors it. And *that* little piece of knowledge is what makes this whole thing work.

### Step 1: Install `keepalived`

We’re going to use **keepalived** to automate the failover. It’ll run our script at regular intervals and switch routes if it detects trouble on the primary connection.

If you don’t have `keepalived` installed yet:

```bash
sudo apt install keepalived
```

### Step 2: Create the keepalived configuration

Now create (or edit) the file at `/etc/keepalived/keepalived.conf`:

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Paste this in:

```bash
vrrp_script chk_downtime {
    script "/etc/keepalived/check_downtime.sh"
    interval 3
    weight +20
}

vrrp_instance FAILOVER {
    state MASTER
    interface wlp2s0
    virtual_router_id 51
    priority 200
    advert_int 2

    track_script {
        chk_downtime
    }

    notify_master "/etc/keepalived/failover.sh"
    notify_backup "/etc/keepalived/failover.sh"
}
```

### Step 3: Write the downtime check script

This little script checks if your primary connection is still alive.

Create the file:

```bash
sudo nano /etc/keepalived/check_downtime.sh
```

Paste:

```bash
#!/bin/bash

logger "check_wlp3s0: Checking internet Access via Wifi"
TARGET="8.8.4.4"
INTERFACE="wlp2s0"
GATEWAY="192.168.1.1"

# Check current route
CURRENT_ROUTE=$(ip route get $TARGET 2>/dev/null)

if ! echo "$CURRENT_ROUTE" | grep -q "dev $INTERFACE"; then
    echo "Route to $TARGET is not via $INTERFACE. Adding route..."
    ip route add $TARGET via $GATEWAY dev $INTERFACE
fi

ping -I wlp3s0 -c 2 -W 1 $TARGET > /dev/null 2>&1

if [ $? -eq 0 ]; then
    echo "Internet is reachable via wlp3s0"
    /etc/keepalived/set_interface.sh
    exit 0
else
    echo "Internet is not rechable via wlp3s0"
    /etc/keepalived/set_interface.sh
    exit 1
fi
```

Don’t forget to give it execute permissions:

```bash
sudo chmod u+x /etc/keepalived/check_downtime.sh
```

### Step 4: Create the failover script

Almost there — now let’s add the script that actually makes the switch.

Create:

```bash
sudo nano /etc/keepalived/failover.sh
```

Paste:

```bash
#!/usr/bin/env bash

# Network Interface Switching Script
# Automatically switches between primary and secondary network interfaces
# based on connectivity tests and manages NAT rules accordingly

# set -o errexit   # Exit on any command failure [3]
# set -o pipefail  # Exit on pipe failure [3]
# set -o nounset   # Exit on undefined variables [3]

# ============================================================================
# CONFIGURATION
# ============================================================================

readonly TARGET_HOST="8.8.4.4"
readonly PRIMARY_INTERFACE="wlp2s0"
readonly SECONDARY_INTERFACE="usbnet0"
readonly PRIMARY_GATEWAY="192.168.1.1"
readonly SECONDARY_GATEWAY_DEFAULT="192.168.225.1"
readonly NAT_SUBNET="10.10.2.0/24"
readonly PRIMARY_METRIC=200
readonly SECONDARY_METRIC=210

# ============================================================================
# UTILITY FUNCTIONS
# ============================================================================

# Print error messages to stderr [10]
error() {
    printf "ERROR: %s\n" "$*" >&2
}

# Print informational messages
info() {
    printf "INFO: %s\n" "$*"
}

# Check if command succeeded [13]
check_command() {
    if ! "$@"; then
        error "Command failed: $*"
        return 1
    fi
}

# ============================================================================
# NETWORK INTERFACE FUNCTIONS
# ============================================================================

# Check if network interface exists and is available
is_interface_available() {
    local interface="$1"
    ip link show "$interface" &>/dev/null
}

# Get gateway for specific interface from routing table
get_interface_gateway() {
    local interface="$1"
    ip route show default | grep "dev $interface" | awk '{print $3}' | head -1
}

# Obtain gateway via DHCP for interface
obtain_dhcp_gateway() {
    local interface="$1"
    local gateway
    
    info "Attempting to obtain gateway via DHCP for $interface..."
    
    # Run dhclient and extract gateway from output [8]
    gateway=$(dhclient -v "$interface" 2>&1 | awk '/DHCPACK/ {print $NF}' | head -1)
    
    if [[ -n "$gateway" ]]; then
        # Remove routes without metrics to avoid conflicts
        local no_metric_routes
        no_metric_routes=$(ip route show default | awk '!/metric/ {print}')
        if [[ -n "$no_metric_routes" ]]; then
            echo "$no_metric_routes" | while read -r route; do
                ip route del $route 2>/dev/null || true
            done
        fi
        echo "$gateway"
    else
        error "Failed to obtain gateway via DHCP"
        return 1
    fi
}

# Get current gateway with lowest metric
get_current_gateway() {
    ip route show default | \
        awk 'BEGIN {min=10000} 
             {if ($0 ~ /default/ && $NF+0 < min) {min=$NF; gw=$3}} 
             END {print gw}'
}

# Test connectivity to target via specific interface
test_connectivity() {
    local interface="$1"
    local target="$2"
    
    ping -I "$interface" -c 2 -W 1 "$target" &>/dev/null
}

# ============================================================================
# ROUTING MANAGEMENT FUNCTIONS
# ============================================================================

# Add specific route if it doesn't exist
ensure_target_route() {
    local target="$1"
    local gateway="$2" 
    local interface="$3"
    
    local current_route
    current_route=$(ip route get "$target" 2>/dev/null || true)
    
    if ! echo "$current_route" | grep -q "dev $interface"; then
        info "Adding route to $target via $interface"
        check_command ip route add "$target" via "$gateway" dev "$interface"
    fi
}

# Flush all default routes and set new ones
configure_routing() {
    local primary_gw="$1"
    local primary_int="$2"
    local secondary_gw="$3"
    local secondary_int="$4"
    local primary_metric="$5"
    local secondary_metric="$6"
    
    info "Configuring routing with primary: $primary_int, secondary: $secondary_int"
    
    # Flush existing default routes [6]
    ip route flush default
    
    # Add primary route
    check_command ip route add default via "$primary_gw" dev "$primary_int" metric "$primary_metric"
    
    # Add secondary route if available
    if [[ -n "$secondary_gw" && -n "$secondary_int" ]]; then
        check_command ip route add default via "$secondary_gw" dev "$secondary_int" metric "$secondary_metric"
    fi
}

# ============================================================================
# NAT/IPTABLES MANAGEMENT FUNCTIONS
# ============================================================================

# Check if iptables rule exists [9]
iptables_rule_exists() {
    local interface="$1"
    iptables -t nat -C POSTROUTING -s "$NAT_SUBNET" -o "$interface" -j MASQUERADE 2>/dev/null
}

# Remove iptables rule if it exists
remove_iptables_rule() {
    local interface="$1"
    
    if iptables_rule_exists "$interface"; then
        info "Removing NAT rule for $interface"
        iptables -t nat -D POSTROUTING -s "$NAT_SUBNET" -o "$interface" -j MASQUERADE
    fi
}

# Add iptables masquerade rule
add_iptables_rule() {
    local interface="$1"
    
    if ! iptables_rule_exists "$interface"; then
        info "Adding NAT rule for $interface"
        iptables -t nat -A POSTROUTING -s "$NAT_SUBNET" -o "$interface" -j MASQUERADE
    fi
}

# ============================================================================
# MAIN LOGIC FUNCTIONS
# ============================================================================

# Initialize secondary interface and get its gateway
setup_secondary_interface() {
    local secondary_gw=""
    
    if is_interface_available "$SECONDARY_INTERFACE"; then
        secondary_gw=$(get_interface_gateway "$SECONDARY_INTERFACE")
        
        info "Secondary Gateway: ${secondary_gw:-"Not found"}"
        
        if [[ -z "$secondary_gw" ]]; then
            secondary_gw=$(obtain_dhcp_gateway "$SECONDARY_INTERFACE" || echo "")
            if [[ -n "$secondary_gw" ]]; then
                info "Obtained secondary gateway: $secondary_gw"
            fi
        fi
    else
        info "Secondary interface $SECONDARY_INTERFACE not found or not connected. Skipping secondary routing."
    fi
    
    echo "$secondary_gw"
}

# Handle primary interface routing and NAT
configure_primary_interface() {
    local current_gw="$1"
    local secondary_gw="$2"
    
    if [[ "$current_gw" != "$PRIMARY_GATEWAY" ]]; then
        info "Switching to: $PRIMARY_INTERFACE"
        
        configure_routing "$PRIMARY_GATEWAY" "$PRIMARY_INTERFACE" \
                         "$secondary_gw" "$SECONDARY_INTERFACE" \
                         "$PRIMARY_METRIC" "$SECONDARY_METRIC"
        
        add_iptables_rule "$PRIMARY_INTERFACE"
    else
        info "Already using $PRIMARY_INTERFACE as default gateway."
        if ! iptables_rule_exists "$PRIMARY_INTERFACE"; then
            add_iptables_rule "$PRIMARY_INTERFACE"
        fi

        remove_iptables_rule "$SECONDARY_INTERFACE"
    fi
}

# Handle secondary interface routing and NAT
configure_secondary_interface() {
    local current_gw="$1"
    local secondary_gw="$2"
    
    if [[ -n "$secondary_gw" && "$current_gw" != "$secondary_gw" ]]; then
        info "Switching to: $SECONDARY_INTERFACE"
        
        configure_routing "$secondary_gw" "$SECONDARY_INTERFACE" \
                         "$PRIMARY_GATEWAY" "$PRIMARY_INTERFACE" \
                         "$SECONDARY_METRIC" "$((PRIMARY_METRIC + 20))"
        
        add_iptables_rule "$SECONDARY_INTERFACE"
    else
        info "No working secondary interface or already using it as default gateway."
        if ! iptables_rule_exists "$SECONDARY_INTERFACE"; then
            add_iptables_rule "$SECONDARY_INTERFACE"
        fi

        remove_iptables_rule "$PRIMARY_INTERFACE"
    fi
}

# Display current routing configuration
show_routing_status() {
    echo ""
    info "Applied following settings:"
    ip route show default
}

# ============================================================================
# MAIN EXECUTION
# ============================================================================

main() {
    info "Starting network interface switching script"
    
    # Setup secondary interface
    local secondary_gw
    secondary_gw=$(setup_secondary_interface)
    
    # Ensure target route exists via primary interface
    ensure_target_route "$TARGET_HOST" "$PRIMARY_GATEWAY" "$PRIMARY_INTERFACE"
    
    # Get current gateway
    local current_gw
    current_gw=$(get_current_gateway)
    info "Current Gateway: $current_gw"
    
    # Test primary interface connectivity [8]
    if test_connectivity "$PRIMARY_INTERFACE" "$TARGET_HOST"; then
        info "Target $TARGET_HOST is reachable via $PRIMARY_INTERFACE"
        configure_primary_interface "$current_gw" "$secondary_gw"
    else
        info "Target $TARGET_HOST is NOT reachable via $PRIMARY_INTERFACE"
        configure_secondary_interface "$current_gw" "$secondary_gw"
    fi
    
    show_routing_status
    info "Network interface switching completed successfully"
}

# Execute main function if script is run directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

Also make sure it’s executable:

```bash
sudo chmod u+x /etc/keepalived/failover.sh
```

And that’s it — you’ve got a basic automated failover system ready to step in whenever your internet tries to betray you. Make sure to change variables at the top of the script according to your situation.

### The Finale: Spreading the Internet Love

Now that our failover setup is up and running like a responsible adult, I figured — why keep all this internet awesomeness to just one machine?  
Let’s share it with the rest of the devices in the house!  
(You're welcome, future smart bulbs, fridges, and random IoT doodads.)

This final part is where we build a virtual network that bridges a physical LAN port on my Ubuntu machine, so I can hook it up to a second router and spread that sweet, sweet internet.

### Step 1: Time to mess with netplan

Ubuntu uses **netplan** for its network configs — basically just YAML files with feelings.  
Let’s create our virtual bridge network.

Fire up your editor:

```bash
sudo nano /etc/netplan/01-bridge.yaml
```

And paste in:

```bash
network:
  version: 2
  ethernets:
    eno0:
      dhcp4: false
    wlp3s0:
      dhcp4: true
    usbnet0:
      dhcp4: true
  bridges:
    vmbr0:
      addresses:
      - "10.10.2.1/24"
      dhcp4: false
      interfaces:
      - eno0
      parameters:
        forward-delay: "0"
        stp: false
```

### Step 2: What this actually does

* Creates a bridge called `vmbr0` using the subnet `10.10.2.0/24`
    
* Bridges it to `eno0` — that’s my physical LAN port
    
* No DHCP server is running on my Ubuntu machine — we’re keeping it simple (and very manual)
    

### Step 3: Connecting devices

Since there’s no DHCP, any device you connect to `eno0` will need to be set up with a **static IP**. For example:

* IP: `10.10.2.2`
    
* Subnet mask: `255.255.255.0`
    
* Gateway: `10.10.2.1` (your Ubuntu machine)
    

And that’s it — your Ubuntu box is now basically moonlighting as a mini-router for your whole house (or lab, or dungeon — I’m not judging).