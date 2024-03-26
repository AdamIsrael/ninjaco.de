---
title: "Hosting static sites on Digital Ocean's App Platform (for free!)"
description: Digital Ocean allows you to host up to three static sites for free. In this post, I explore their App Platform in order to automatically deploy this Hugo-based blog from a Github repository.
date: 2024-03-23T15:02:06-04:00
image:
math:
license:
hidden: false
comments: true
draft: false
categories:
- Technical
tags:
- Digital Ocean
- hugo
---

I've been using [Dreamhost](https://www.dreamhost.com/) to host most of my domains, including a couple of [Hugo-based](https://gohugo.io) static websites. I also have a [Digital Ocean](https://www.digitalocean.com/) (DO) account where I've been running a droplet for the past decade. I recently discovered their [App Platform](https://www.digitalocean.com/products/app-platform), which allows you to deploy up to three static sites for free. While my current hosting is adequate, it was a bit of a process to setup the workflows to automate publishing the _static_ content when I merged in changes to its Github repo.

Disclaimer: I work at Digital Ocean. Opinions are my own and _totally_ not influenced by my employment.

> [Hugo](https://gohugo.io/) is a static site generator. You write your content in Markdown and it's published as HTML. The Digital Ocean App Platform has built-in support for Hugo sites but it's [buildpack](https://docs.digitalocean.com/products/app-platform/reference/buildpacks/hugo/) doesn't currently support [Hugo Modules](https://gohugo.io/categories/hugo-modules/). The alternative is to use a `Dockerfile`, which is the path we'll explore here.


The root of [this Hugo site](https://github.com/AdamIsrael/ninjaco.de) looked like this:

```console
.
├── LICENSE
├── README.md
├── assets
├── config
├── content
├── go.mod
├── go.sum
├── layouts
├── public
├── resources
└── static
```

DO's App Platform uses buildpack(s) are used to detect the type of application you're trying to deploy. The Hugo buildpack, for example, looks for a `config.[json|toml|yaml]` file. That works with some sites and versions of Hugo but didn't correctly identify Hugo for this site because it doesn't recognize the [configuration directory](https://gohugo.io/getting-started/configuration/#configuration-directory). It does, however, detect the `go.sum` used by Hugo Modules and assumeed that this is a Go project. The build will then fail because there's no Go source code to compile.

After a bit of poking around, I discovered that the platform could also build and deploy from a `Dockerfile`, as per this [solution](https://discourse.gohugo.io/t/issues-with-deploying-on-do-app-platform/44872/5). I added this `Dockerfile` to the root of my repository:

```docker
FROM peaceiris/hugo:latest-mod

WORKDIR /app
COPY . /app
RUN hugo -d public
```

> This comes from [hugo-extended-docker](https://github.com/peaceiris/hugo-extended-docker), which is a Docker image containing Hugo extended and Hugo modules. It allows you to build the Hugo site inside a container, avoiding the use of the Hugo buildpack.

Once that file is committed and pushed, I was ready to [begin](https://cloud.digitalocean.com/apps/new).

![Create Resource from Source Code](create-resource.png)

The platform will detect two different applications: the Dockerfile and a Go application.

![Auto-detected applications](app.png)

Delete the second app (`ninjaco.de2`). That maps to the Go application. We'll focus our attention on the Docker container. Edit that application and change the Resource Type from Web Service to Static Site. Set the Output Directory to `/app/public`, which is where we've built our static content. Finally, click the link to go back.

This will change the hosting plan to Starter, which allows for three free static websites to be hosted.

![The correct application](app2.png)

Click next through `Environment`.

On the Info page, I've edited the application page to `ninja-code` so it's more visible in the platform's dashboard.

![Settings](settings.png)

Click next and review. The Monthly App Cost should be $0.00.

Click `Create Resources`. This will trigger a pull of your repo. The Docker buildpack will run the Dockerfile, which generates your static site, and upload the content from the `public` directory. This may take several minutes. When it's complete, go to the [Apps dashboard](https://cloud.digitalocean.com/apps/) to see your static site's status. Follow the Live App URL and you should see your static site.

Now, when new content gets merged to `main`, it will automatically trigger a new build and deployment. Within minutes, your static site will be updated.

The last step was to change the DNS to point to the new site. Go into the App Settings and edit the Domains.

Click "Add Domain" and enter your domain name. This will give you the option to allow Digital Ocean to manage the domain's DNS, or you can manage it manually. I've opted to move the DNS from Dreamhost to Digital Ocean.

Repeat this process to add any subdomains (like www.ninjaco.de).

Once I changed the DNS (and waited patiently for them to propogate), the new site was live and you can read this post.

It wasn't a flawless process, but using the Dockerfile to build and publish the site proved to be a handy workaround to using the native Hugo buildpack. If you have a static website you want to host and don't feel like spinning up a VM or using shared hosting, this gives you a straightforward path to getting up and running.
