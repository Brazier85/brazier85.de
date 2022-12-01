---
title: "How this page is build"
date: 2022-12-01T10:46:55+01:00
draft: false
---

This pages is build with [hugo](https://gohugo.io/) with the [bilberry-hugo-theme](https://github.com/Lednerb/bilberry-hugo-theme) and hosted on [all-inkl.com](https://all-inkl.com).

## Why hugo

In the last few years I tried serveral cms-applications like Wordpress, Joomla, Typo3, ect. but all of them where to comiplicated to maintain. So I searched for something simple but also "good looking". After some google-searches I read about the idea to use static-pages for a personal blog.

Then I tried some generators like [jekyll](https://jekyllrb.com/) or [gatsby](https://www.gatsbyjs.com/) but they did not work right away and did require alot of setup. Then I found [hugo](https://gohugo.io/) and simply typed `brew install hugo` into my console and it worked.

For the comment system I use [giscus](https://giscus.app/de).

### How it is deployed

The raw content of this page is on [github](https://github.com/Brazier85/brazier85.de). There I use a github-action to deploy the build pages to my personal webspace. The github-action looks like this:

```yaml
name: deploy to all-inkl

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    steps:
      - name: Fetch submodules
        uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: andreiio/rclone-action@v1
        env:
          RCLONE_CONFIG_ALLINKL_TYPE: ftp
          RCLONE_CONFIG_ALLINKL_HOST: ${{ secrets.RCLONE_CONF_SERVER }}
          RCLONE_CONFIG_ALLINKL_USER: ${{ secrets.RCLONE_CONF_USER }}
          RCLONE_CONFIG_ALLINKL_PASS: ${{ secrets.RCLONE_CONF_PASSWORD }}
        with:
          args: sync public ALLINKL:/
```