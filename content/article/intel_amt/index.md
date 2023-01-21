---
title: "Intel AMT"
date: 2023-01-21T20:35:04+01:00
draft: false

categories: ["intel","amt"]
tags: ["proxmox","server"]
toc: false
author: "Ferdinand Berger"
---

Do you know about the free and builtin intel KVM on your server? A few days ago I moved my infrastructure to a new "server" (Lenovo M900) and I wanted to add a KVM solution be able to get access to the pc without adding a monitor to my rack.

<!--more-->

## Intel AMT or vPro

Intel AMT (Active Management Technology) makes remote maintenance of desktop PCs possible. AMT has been integrated with certain Intel chipsets. It even includes the ability to use VNC for a remote desktop session. More can be found on [wikipedia](https://en.wikipedia.org/wiki/Intel_Active_Management_Technology)

## Setup

You have to activate AMT in your BIOS settings - it should be in the advanced settings menu:

- Advanced > AMT Configuration

Before you can use it you must restart your PC and configure it. You must press `Ctrl + P` while your system is starting up and then select `MEBx Login` and login with the credentials `admin:admin`. Next you can turn on the features you need in your environment. In my case I added a static IP-address and activated `SOL/Storage Redirection/KVM`.

## Usage

After setting everything up you can visit the webinterface via http://IP-ADDRESS:16992/ (port 16992). To use the remote desktop function you have to insert a "video dummy device" into your video port. Without this device the graphics card will fo into sleep/idle mode. I bought this one [here](https://amzn.to/3XQ0IBp) on amazon.

There are few different software tools that you can use to access AMT, but the one that I found most useful is [MeshCommander](https://www.meshcommander.com/home), an remote management tool that supports many OOB features, including remote desktop, remote terminal, and remote access to files. It runs on all of the common platforms, including Windows, Linux, and macOS.