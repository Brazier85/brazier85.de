---
title: "How this page is built"
date: 2022-12-01T10:46:55+01:00
draft: false

categories: ["code"]
tags: ["hugo", "github", "pipeline", "all-inkl", "deployment"]
toc: false
author: "Ferdinand Berger"
---

This page is built with [hugo](https://gohugo.io/) and the [bilberry-hugo-theme](https://github.com/Lednerb/bilberry-hugo-theme) with some custom changes. It is hosted on [all-inkl.com](https://all-inkl.com).

<!--more-->

## Why hugo?

In the last few years I tried several cms-applications like Wordpress, Joomla, Typo3, ect. but all of them where too complicated to maintain. So I searched for something simple but also "good looking". After some google-searches I read about the idea to use static-pages for a personal blog.

Then I tried some generators like [jekyll](https://jekyllrb.com/) or [gatsby](https://www.gatsbyjs.com/) but they did not work right away and also require a lot of work to setup and maintain. Then I found [hugo](https://gohugo.io/) and simply typed `brew install hugo` into my console and it worked.

For the commenting system I use [giscus](https://giscus.app/) and [algolia](https://www.algolia.com/) as search engine..

## How it is deployed

The raw content of this page is on [github](https://github.com/Brazier85/brazier85.de). There I use a github-action to built and deploy the pages to my personal webspace. The github-action looks like this:

{{< code type="yaml" title="deploy.yml" >}}
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

      - name: built
        run: hugo --minify

      - name: Deploy
        uses: andreiio/rclone-action@v1
        env:
          RCLONE_CONFIG_ALLINKL_TYPE: ftp
          RCLONE_CONFIG_ALLINKL_HOST: ${{ secrets.RCLONE_CONF_SERVER }}
          RCLONE_CONFIG_ALLINKL_USER: ${{ secrets.RCLONE_CONF_USER }}
          RCLONE_CONFIG_ALLINKL_PASS: ${{ secrets.RCLONE_CONF_PASSWORD }}
          RCLONE_FTP_CONCURRENCY: 10
          RCLONE_TRANSFERS: 5
          RCLONE_CHECKERS: 5
        with:
          args: sync public ALLINKL:/

      - name: Upload Search Data
      working-directory: ./algolia # equivalent of 'cd algolia'
      run: |
        npm install
        npm run data-upload -- -c \
          -f ../public/index.json \
          -a TUHYJRF01B \
          -k ${{ secrets.ALGOLIA_ADMIN_API_KEY }} \
            -n prod_brazier85 
{{< /code >}}

## Changes I made to the theme

**Note:** The exact position of these changes you can find on [github](https://github.com/Brazier85/brazier85.de).

### Added donation image and link
{{< code type="yaml" title="config.toml" >}}
showDonation = true
donationImage = "/images/kofi_bg_tag_white.png"
donationLink = "https://ko-fi.com/brazier85"
{{< /code >}}

{{< code type="yaml" title="layouts/partials/footer.html" >}}
  {{ if .Site.Params.showDonation | default true }}
      <div class="donation">
          <a href="{{ .Site.Params.donationLink }}" target="_blank" >
              <img src="{{ .Site.Params.donationImage }}" />
          </a>
      </div>
  {{ end }}
{{< /code >}}


### External links with _blank
{{< code type="html" title="layouts/_default/_markup/render-link.html" >}}
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination "http" }} target="_blank"{{ end }}>{{ .Text }}</a>
{{< /code >}}

### Collapsible code blocks
For this feature you should divinely check out my code on [github](https://github.com/Brazier85/brazier85.de). There are a few things you have to change. First, there are custom css and js files you have to include.
{{< code type="js" title="static/js/coll_code.js" >}}
// Collapsible Hugo code blocks
// by Jiri De Jagere, @JiriDJ
// modified by Ferdinand Berger @Brazier85

var height = "300px";

if (
  document.readyState === "complete" ||
    (document.readyState !== "loading" && !document.documentElement.doScroll)
) {
  makeCollapsible();
} else {
  document.addEventListener("DOMContentLoaded", makeCollapsible);
}

function toggle(e) {
  e.preventDefault();
  var link = e.target;
  var div = link.parentElement.parentElement;

  if (link.innerHTML == "show more&nbsp;") {
    link.innerHTML = "show less&nbsp;";
    div.style.maxHeight = "";
    div.style.overflow = "none";
    link.parentElement.style.bottom = "15px";
  }
  else {
    link.innerHTML = "show more&nbsp;";
    div.style.maxHeight = height;
    div.style.overflow = "hidden";
    link.parentElement.style.bottom = "0px";
    div.scrollIntoView({ behavior: 'smooth' });
  }
}

function makeCollapsible() {
  var divs = document.querySelectorAll('.highlight-wrapper');

  for (i=0; i < divs.length; i++) {
    var div = divs[i];
    if (div.offsetHeight > parseInt(height, 10)) {
      div.style.maxHeight = height;
      div.style.overflow = "hidden";

      var e = document.createElement('div');
      e.className = "highlight-link";

      var html = '<a href="">show more&nbsp;</a>';
      e.innerHTML = html;
      e.style.bottom = "0px";
      div.appendChild(e);
    }
  }

  var links = document.querySelectorAll('.highlight-link');
  for (i=0; i<links.length; i++) {
    var link = links[i];
    link.addEventListener('click', toggle);
  }
}
{{< /code >}}
{{< code type="css" title="static/css/coll_code.css" >}}
.highlight-before {
    padding: 4px 0px 0px 4px;
    margin-bottom: -15px;
}

.highlight-wrapper {
    position: relative;
    background-color: #157EAC;
    color: #fff;
    border: 4px solid #157EAC;
    border-radius: 4px;
    margin-bottom: 15px;
}

.highlight-link {
    position: absolute;
    right: 0;
}

.highlight-link a {
    color: #fff !important;
}

.highlight {
    margin-bottom: -24px !important;
}
{{< /code >}}

And you need this `shotcode` to render it properly
{{< code type="yaml" title="layout/shortcodes/code.html" >}}
{{- $type := (.Get "type" | default (.Get 0)) }}
{{- $title := (.Get "title" | default (.Get 1)) }}

<div class="highlight-wrapper">
    <div class="highlight-before">{{ $title }}</div>
        {{ highlight (trim .Inner "\n\r") ($type) "" }}
</div>
{{< /code >}}