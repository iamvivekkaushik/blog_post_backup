---
title: "Guide: How to configure Proxmox with WiFi"
datePublished: Tue Dec 31 2024 18:06:20 GMT+0000 (Coordinated Universal Time)
cuid: cm5cs51gx002i09l4hg3tdkwf
slug: guide-how-to-configure-proxmox-with-wifi
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735664040078/a5455883-992b-46c6-8624-027f2cdcdea7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1735668373877/5bcd1d60-dcfc-492b-b961-11a0f374f79c.png
tags: dns, linux, wifi, networking, proxmox

---

So, one day I saw my old laptop sitting there, collecting dust like it was auditioning for a role in an apocalypse movie. And then it hit me: "Why not turn it into a server?" There was just one minor hiccup. Its Ethernet port was as dead as my dreams of becoming a professional guitarist. But hey, the WiFi was still kicking! After hours scouring the internet and performing digital sorcery, I made it work. Here's a guide for anyone crazy enough to try the same.

### Step 1: Configure WiFi Credentials

First things first—let's get that server connected to WiFi. Use the following magical incantation:

```bash
wpa_passphrase SSIDNAME PASSWORD >> /etc/wpa_supplicant/wpa_supplicant.conf
```

Replace `SSIDNAME` and `PASSWORD` with your network's details. Then, edit the resulting file at `/etc/wpa_supplicant/wpa_supplicant.conf` to look something like this:

```apache
ctrl_interface=/run/wpa_supplicant
update_config=1
country=IN

network={
        ssid="SSID_NAME"
        #psk="asdflasf"
        psk=89d64643245wsfdfg237ed3ec3ef31c47e75430f0asdfji40598
        proto=WPA RSN
        key_mgmt=WPA-PSK
}
```

Pro tip: That `#psk="your_password_in_plain_text"` line is commented out for a reason. Don't let your server broadcast your WiFi password to the world!

### Step 2: Configure Network Interfaces

Next up, let’s tackle the network interfaces. Open your favorite text editor (or `nano`, if you must) and edit `/etc/network/interfaces` like so:

```apache
auto lo
iface lo inet loopback

# Wifi interface autoconnect using wpa_supplicant.conf
auto wlp3s0
iface wlp3s0 inet dhcp
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

# Virtual bridge network
auto vmbr0
iface vmbr0 inet static
        address 10.1.1.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s '10.1.1.0/24' -o wlp3s0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.1.1.0/24' -o wlp3s0 -j MASQUERADE
        post-up iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1

source /etc/network/interfaces.d/*
```

**Heads up:** Your WiFi interface might not be called `wlp3s0`. Use the `ip a` command to find out what it’s really called. Also, if you want to use a different subnet, replace `10.1.1.0/24` with your preferred network range.

If you’re feeling fancy, you can even change your MAC address with the following configuration:

```apache
iface wlp3s0 inet static
        address 192.168.1.2
        netmask 255.255.255.0
        gateway 192.168.1.1
        dns-nameservers 1.1.1.1 1.0.0.1
        hardware ether 06:61:D1:65:61:64
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

When you’re done, restart the networking service to apply your changes:

```bash
systemctl restart networking
```

### Step 3: Install and Configure `dnsmasq`

Now, let’s set up a DHCP server to dish out IP addresses to your VMs connected to `vmbr0`. Start by installing `dnsmasq`:

```bash
apt install dnsmasq
```

Then, edit its configuration file `/etc/dnsmasq.conf` with the following lines:

```bash
# Set the interface where you want DHCP server to assign IP
interface=vmbr0

# DHCP address range
dhcp-range=10.1.1.1,10.1.1.255,255.255.255.0,12h
```

Restart `dnsmasq` to apply the changes:

```bash
systemctl restart dnsmasq
```

### Conclusion

Congratulations, you’ve successfully turned a dusty old laptop with a broken Ethernet port into a WiFi-powered homelab server!

### Bonus: Configure Lid Configuration

If you’re also using a laptop for this, you might want to keep it going even if the lid closes! Edit the `/etc/systemd/logind.conf` file to make sure it stays alive even when you close the lid:

```bash
HandleSuspendKey=ignore
HandleSuspendKeyLongPress=ignore

HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore

LidSwitchIgnoreInhibited=no 
```

After updating the above file with the values provided restart \`logind\`:

```bash
systemctl restart systemd-logind
```

### One More Thing!

With this setup, your virtual machines have full access to your router’s network. However, devices directly connected to the router won’t know how to reach the network you just defined. To fix this, you’ll need to create a route entry in your router’s or device’s routing table.

Since router interfaces vary wildly, here’s how you can do it on a Mac:

```bash
sudo route -n add 10.1.1.0/24 192.168.1.2
```

Just replace `10.1.1.0/24` with your subnet and `192.168.1.2` with your homelab server’s IP address. In case you want to remove it use:

```bash
sudo route -n delete 10.1.1.0/24 192.168.1.2
```