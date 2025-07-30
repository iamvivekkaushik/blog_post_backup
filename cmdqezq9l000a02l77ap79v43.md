---
title: "Keeping Dokku Apps Fresh: The Easy Way to Update Deployed Images"
datePublished: Wed Jul 30 2025 20:24:42 GMT+0000 (Coordinated Universal Time)
cuid: cmdqezq9l000a02l77ap79v43
slug: keeping-dokku-apps-fresh-the-easy-way-to-update-deployed-images
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1753906975717/9f1fd862-f410-4089-805e-dff5add6085f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1753907074559/3cdbab24-6b75-4030-8183-a100f6a6d9bc.png
tags: dokku

---

Ever found yourself in the endless cycle of "delete Docker image, redeploy, repeat" to keep your Dokku apps updated? You’re not alone. If your Dokku-deployed application relies directly on a Docker image (perhaps from Docker Hub or your private registry), you might've noticed a frustrating behavior: once Dokku pulls the image even when you use the `latest` tag it won’t fetch updates unless you manually delete the old image.  
This got me scratching my head until I stumbled onto an elegant fix that streamlines the workflow and fits perfectly into the minimalist, automation-loving philosophy I share here.

## The Problem

By default, Dokku holds onto the Docker image it fetched during the initial deploy. No matter how many times you rebuild or redeploy with the same tag, Dokku won’t pull a fresher image from the registry unless you interfere by deleting it manually.

## The Solution: Automate Image Updates with Build Options

Dokku’s flexibility shines if you know where to look. The trick is to tell Dokku to pull the latest image from your registry **every time you rebuild**—completely bypassing the need to manually delete old images.

## Step 1: Add Docker Build Options to Your App

You'll use Dokku's `docker-options` to modify how the build and deploy steps work for your app.

```bash
dokku docker-options:add <app-name> build --pull --no-cache
```

* Replace `<app-name>` with the name of your Dokku app.
    
* `--pull` makes sure Docker pulls the latest version of the image from your registry.
    
* `--no-cache` ensures Docker doesn’t use a cached image locally.
    

## Step 2: Rebuild Your App

Now, whenever you want to update to the latest image (say, after pushing to your Docker registry), run:

```bash
dokku ps:rebuild <app-name>
```

That's it! Dokku will pull down the freshest image, rebuild, and restart your app—no more manual deletions required.

## That’s a Wrap

This approach works wonders for keeping your Dokku deployments lean and up-to-date.