---
title: "Configure qBittorrent on Dokku and Mount SMB NAS for Download Storage"
seoTitle: "Configure qBittorrent on Dokku and Mount SMB NAS for Download Storage"
seoDescription: "Set up qBittorrent on Dokku with nas connected as storage media."
datePublished: Sun May 19 2024 09:55:23 GMT+0000 (Coordinated Universal Time)
cuid: clwdd35jd00030al3evvl90za
slug: configure-qbittorrent-on-dokku-and-mount-smb-nas-for-download-storage
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1716112355652/0bfc8576-859c-4f78-8722-1394f9df5211.png
tags: dokku, torrent, nas, smb, qbittorrent

---

Ready to set up qBittorrent on Dokku? Follow these steps, and you'll be up and running in no time! (This article assumes you have dokku and a nas server already up and running.)

#### Step 1: Create a New App

First, create your new app in Dokku with the following command:

```bash
dokku apps:create qbittorrent
```

Feel free to replace `qbittorrent` with your preferred app name.

#### Step 2: Mount a Directory for qBittorrent Configuration

Next, mount a directory for qBittorrent's configuration:

```bash
dokku storage:mount qbittorrent /home/dokku/qbittorrent/data:/config
```

#### Step 3: Create a Docker Volume

Now, let's create a volume. Run the following command, replacing the placeholders with your specific values:

```bash
docker volume create --driver local --opt type=cifs --opt device=//<nas_url_here>/<nas_path> --opt o=username=<your_username>,password=<your_password>,port=<your_port>,uid=1000,gid=1000,forceuid <name_of_the_volume>
```

Replace `<nas_url_here>`, `<nas_path>`, `<your_username>`, `<your_password>`, `<your_port>`, and `<name_of_the_volume>` with your actual NAS details and desired volume name.

#### Step 4: Mount the Volume

Mount the newly created volume:

```bash
dokku storage:mount qbittorrent <name_of_the_volume>:/downloads
```

Ensure you replace `<name_of_the_volume>` with the volume name you used earlier.

#### Step 5: Set Environment Variables

Configure the environment variables for your app:

```bash
dokku config:set qbittorrent PGID=1000 PUID=1000 TZ=Asia/Kolkata WEBUI_PORT=8080
```

#### Step 6: Deploy the Docker Image

Initiate the deployment using the docker image:

```bash
dokku git:from-image qbittorrent linuxserver/qbittorrent:4.6.0
```

#### Step 7: Configure Ports

Set up the port configuration to make your app accessible via Dokku's proxy server:

```bash
dokku ports:set qbittorrent http:80:8080
```

#### Step 8: Set a Domain

Finally, set a domain for your app:

```bash
dokku domains:add qbittorrent <your_domain_here>
```

Replace `<your_domain_here>` with your actual domain.

---

And that's it! Your qBittorrent app should now be up and running on Dokku. Happy torrenting!