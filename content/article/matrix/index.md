---
title: "Matrix"
date: 2023-01-07T21:35:49+01:00
draft: true

categories: ["Docker", "Messaging"]
tags: ["Matrix", "WhatsApp", "Telegram", "Signal", "Element"]
toc: false
author: "Ferdinand Berger"
---

The best way to describe [matrix](https://matrix.org/) should be "One app to rule them all". That is basically what it does. In my setup I can use WhatsApp, Telegram and Signal with a single app.

<!--more-->

## What is matrix

## How to get started (docker version)

[GitHub](https://github.com/devture/matrix-synapse-shared-secret-auth/blob/master/shared_secret_authenticator.py)

## Bridges

Bridges in Matrix are used to connect services like WhatsApp to your matrix server. You can find a list of them on the official matrix [site](https://matrix.org/bridges/). Currently I use the following bridges:

- Signal
- Telegram
- WhatsApp

All the bridges are running in `double puppeting` mode. This mode allow me to still use the original apps with sync for all the messages. Without this mode you will not see the messages you send via other clients (eg. Telegram app) in matrix.

The setup is the same for all bridges:

1. Add the bridge to your docker-compose file
2. Run the bridge to generate config files
3. Change the config files
4. Run the bridge to generate a registration file
5. Copy the registration file to your matrix server
6. Restart your matrix server
7. Start the bridge

Here you can find the changes I made to the configuration files:

### WhatsApp

{{< code type="yaml" title="config.yml" >}}
# Homeserver details.
homeserver:
    address: https://matrix.berger-em.net
    domain: matrix.berger-em.net

# Application service host/registration related details
# Changing these values requires regeneration of the registration.
appservice:
    address: http://matrix-whatsapp:29318
    
    # Database config.
    database:
        type: postgres
        uri: postgres://synapse:xxxxxxxxxxxxxxx@matrix-db/whatsapp?sslmode=disable
        # Maximum number of connections. Mostly relevant for Postgres.

# Bridge config
bridge:
    displayname_template: '{{if .FullName}}{{.FullName}}{{else if .PushName}}{{.PushName}}{{else if .BusinessName}}{{.BusinessName}}{{else}}{{.Jid}}{{end}} (WA)'

    login_shared_secret_map:
        matrix.berger-em.net: xxxxxxxxxx

    permissions:
        "*": relay
        "matrix.berger-em.net": user
        "@adminuser:matrix.berger-em.net": admin
{{< /code >}}

### Telegram

{{< code type="yaml" title="config.yml" >}}
# Homeserver details
homeserver:
    address: https://matrix.berger-em.net
    domain: matrix.berger-em.net

# Application service host/registration related details
# Changing these values requires regeneration of the registration.
appservice:
    address: http://matrix-telegram:29317

    database: postgres://synapse:xxxxxxxxxxxxxxxxxxxxxxx@matrix-db/telegram?sslmode=disable

# Bridge config    
bridge: 
    login_shared_secret_map:
        matrix.berger-em.net: xxxxxxx
    
    permissions:
        '*': relaybot
        matrix.berger-em.net: user
        '@adminuser:matrix.berger-em.net': admin

{{< /code >}}

### Signal

{{< code type="yaml" title="config.yml" >}}
# Homeserver details
homeserver:
    address: https://matrix.berger-em.net
    domain: matrix.berger-em.net

# Application service host/registration related details
# Changing these values requires regeneration of the registration.
appservice:
    address: http://matrix-signal:29328

    database: postgres://synapse:xxxxxxxxxxxxxxx@matrix-db/signal?sslmode=disable
    
# Bridge config
bridge:
    login_shared_secret_map:
        matrix.berger-em.net: xxxxxxxxxxxxx
    
    permissions:
        '*': relay
        matrix.berger-em.net: user
        '@adminuser:matrix.berger-em.net': admin

{{< /code >}}

## Problems

This is not really a problem but I can only use matrix while connected to my vpn. This is intended so no one can connect to my internal network.

## Docker-Compose file
{{< code type="yaml" title="docker-compose.yml" >}}
version: "3.4"

services:

  synapse:
    image: docker.io/matrixdotorg/synapse:latest
    container_name: matrix-synapse
    # Since synapse does not retry to connect to the database, restart upon
    # failure
    restart: unless-stopped
    # See the readme for a full documentation of the environment settings
    # NOTE: You must edit homeserver.yaml to use postgres, it defaults to sqlite
    environment:
      - SYNAPSE_CONFIG_PATH=/data/homeserver.yaml
    volumes:
      - /data/docker/matrix/synapse/data:/data
      - /data/docker/matrix/synapse/bridges:/bridges
      # This file is required for double puppeting
      # Get it here: https://github.com/devture/matrix-synapse-shared-secret-auth/blob/master/shared_secret_authenticator.py
      - /data/docker/matrix/synapse/shared_secret_authenticator.py:/usr/local/lib/python3.9/site-packages/shared_secret_authenticator.py
    networks:
      - matrix
      - proxy
    depends_on:
      - db
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - traefik.enable=true
      - traefik.http.services.matrix.loadbalancer.server.port=8008
      - traefik.http.routers.matrix.rule=Host(`matrix.berger-em.net`)
      - traefik.http.routers.matrix.entrypoints=websecure
      - traefik.docker.network=proxy

  db:
    image: docker.io/postgres:12-alpine
    container_name: matrix-db
    networks:
      - matrix
    environment:
      - POSTGRES_DB=synapse
      - POSTGRES_USER=synapse
      - POSTGRES_PASSWORD=xxxxxxxxxxxxxxxxxxxx
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8 --lc-collate=C --lc-ctype=C
    volumes:
      - /data/docker/matrix/db/schemas:/var/lib/postgresql/data
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  # Element is not required!
  # You can use any app you like
  element:
    image: vectorim/element-web:latest
    container_name: matrix-element
    restart: unless-stopped
    volumes:
      - /data/docker/matrix/element/element-config.json:/app/config.json
    networks:
      - matrix
      - proxy
    depends_on:
      - synapse
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - traefik.enable=true
      - traefik.http.services.element.loadbalancer.server.port=80
      - traefik.http.routers.element.rule=Host(`element.berger-em.net`)
      - traefik.http.routers.element.entrypoints=websecure
      - traefik.docker.network=proxy

  # Whatsapp
  mautrix-whatsapp:
    container_name: matrix-whatsapp
    image: dock.mau.dev/mautrix/whatsapp:latest
    restart: unless-stopped
    depends_on:
      - synapse
    volumes:
      - /data/docker/matrix/whatsapp:/data
    networks:
      - matrix
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  # Signal
  mautrix-signal:
    container_name: matrix-signal
    image: dock.mau.dev/mautrix/signal
    restart: unless-stopped
    volumes:
      - /data/docker/matrix/signal/data:/data
      - /data/docker/matrix/signal/signald:/signald
    depends_on:
      - signald
    networks:
      - matrix
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  signald:
    container_name: matrix-signald
    image: docker.io/signald/signald
    restart: unless-stopped
    volumes: 
      - /data/docker/matrix/signal/signald:/signald
    networks:
      - matrix
    depends_on:
      - synapse
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  # Telegram
  mautrix-telegram:
    container_name: matrix-telegram
    image: dock.mau.dev/mautrix/telegram:latest
    restart: unless-stopped
    volumes:
      - /data/docker/matrix/telegram:/data
    networks:
      - matrix
    depends_on:
      - synapse
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

networks:
  matrix:
    name: matrix
  proxy:
    external: true
{{< /code >}}