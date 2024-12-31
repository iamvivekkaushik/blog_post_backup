---
title: "Bring your app online using SSH with a custom domain for free"
datePublished: Sun Jan 29 2023 20:57:39 GMT+0000 (Coordinated Universal Time)
cuid: cldhv5cuk000009mjg0k5d891
slug: bring-your-app-online-using-ssh-with-a-custom-domain-for-free
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1675022773267/4e9fd6d5-ca31-40c3-a6c8-262386e37f3a.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1675025841423/57c97c2c-6715-44d0-9eae-460b048feee6.jpeg
tags: proxy, ngrok, linux, ssh

---

If you're a developer you must've encountered a situation where localhost just wouldn't work for you, maybe you wanted to show off your project to someone else or needed to set up a public URL for a webhook. There are countless situations where you want your project to be accessible from the internet while it is still in the development phase.

What if I told you there is a way to do this with tools you're already using?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675023961031/28aad354-957a-4d42-b4e9-1679c6d9cf15.gif align="center")

Yeah, you read that right, you can bring your application online using SSH.

Why go through the effort when there are free third-party services like ngrok?

The problem with ngrok or any third-party services is that they don't provide a dedicated domain, meaning every time you restart your ngrok client it will give you a different domain and you'll have to change the configuration all over again, in all fairness you could buy a paid plan to have a custom domain setup as well but let's try it my way.

Before we begin, as the title suggests this is not entirely free. I know I said free and it is to some extent, but it is only free if you're already having a domain and a server. A domain may not be a fancy one but can be acquired for free from [Freenom](https://www.freenom.com/en/index.html?lang=en). And there are free tiers available from different cloud providers.

### Setting up Nginx for reverse proxy

Once you have got a server up and running in the cloud, ssh into the server and install Nginx.

```bash
sudo apt install nginx
```

After the successful installation head into the `/etc/nginx/sites-avaialble` directory and create a new server configuration using `sudo touch tunnel.conf` then add the following configuration.

```bash
server {
    listen 80;
    listen [::]:80;

    server_name tunnel.yourdomain.com;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Accept-Encoding gzip;
    }
}
```

replace `tunnel.yourdomain.com` with your domain. Also, add `A record` that points to your server IP for the same.

You can also use [Certbot](https://certbot.eff.org/) to get an SSL certificate for your domain.

### Configure SSH client to port forward

Now that nginx is listening on the server, every time you want to bring your application online just run the following command and access it from your specified domain.

```bash

ssh -R remote_port:host:host_port -i ~/.ssh/private_key user@yourserver.com
```

change the `remote_port` to `8080`, `host` to `localhost` & `host_port` to whatever your application is listening on.

Also, your ssh connection may get disconnected if there is no activity in a while, to keep the connection alive add `ServerAliveInterval 60` at the end of `~/.ssh/config` file.