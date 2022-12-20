---
title: "Basic Smart Home Setup"
date: 2022-12-02T20:20:00+01:00
draft: false

categories: ["Smart Home", "code"]
tags: ["Home Assistant", "ESPHome", "Zigbee2MQTT", "Traefik", "Homegear", "Signal-Cli-Rest-Api", "MQTT", "Smart Home"]
toc: false
author: "Ferdinand Berger"
---

In this article I will explain the basic setup for our/my Smart Home system. In the future there may be some additional articles for a more detailed view on some components.  

<!--more-->

## Components

My Smart Home is based on docker containers running the following applications:
- [Home Assistant](https://home-assistant.io)
- [ESPHome](https://esphome.io/)
- [MariaDB](https://mariadb.org/)
- [Zigbee2MQTT](https://www.zigbee2mqtt.io/)
- [Homegear](https://homegear.eu/) (For Homematic and Intertechno)
- [Mosquitto](https://mosquitto.org/) (MQTT broker)
- [Signal-Cli-Rest-Api](https://github.com/bbernhard/signal-cli-rest-api)
- [Traefik](https://traefik.io/) (Not only in use for my Smart Home)

All my containers are started and maintained with docker-compose.
{{< code type="yaml" title="docker-compose.yml" >}}
version: '3'

services:

# HomeGear
  homegear:
    container_name: homegear
    image: homegear/homegear:latest
    ports:
      - 2001:2001
      - 2002:2002
      - 2003:2003
      - 2004:2004
    environment:
      - TZ=Europe/Berlin
    restart: always
    volumes:
      - /data/docker/home-assistant/homegear/etc:/etc/homegear
      - /data/docker/home-assistant/homegear/lib:/var/lib/homegear
      - /data/docker/home-assistant/homegear/log:/var/log/homegear
    devices:
      - "/dev/serial/by-id/usb-busware.de_CUL433-if00:/dev/ttyACM0"
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"
      - traefik.enable=true
      - traefik.http.services.homegear.loadbalancer.server.port=2001
      - traefik.http.routers.homegear.rule=Host(`homegear.berger-em.net`)
      - traefik.http.routers.homegear.entrypoints=websecure
      - traefik.docker.network=proxy
    networks:
      IoT:
        ipv4_address: 192.168.20.11
      proxy:

# Zigbee2MQTT
  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:latest
    container_name: zigbee2mqtt
    restart: unless-stopped
    volumes:
      - /data/docker/home-assistant/zigbee2mqtt:/app/data
      - /run/udev:/run/udev:ro
    environment:
      - TZ=Europe/Berlin
    devices:
      - "/dev/serial/by-id/usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0:/dev/ttyUSB0"
    networks:
      IoT:
        ipv4_address: 192.168.20.12
      proxy:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"
      - traefik.enable=true
      - traefik.http.services.zigbee.loadbalancer.server.port=8080
      - traefik.http.routers.zigbee.rule=Host(`zigbee.berger-em.net`)
      - traefik.http.routers.zigbee.entrypoints=websecure
      - traefik.docker.network=proxy
    depends_on:
      - mqtt

# MariaDb
  mariadb:
    image: mariadb:10.3
    container_name: mariadb
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: ha_db
      MYSQL_USER: homeassistant
      MYSQL_PASSWORD: "${HA_MYSQL_PASSWORD}"
    volumes:
      # Local path where the database will be stored.
      - /data/docker/home-assistant/mariadb:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      IoT:
        ipv4_address: 192.168.20.13

# Mosquitto
  mqtt:
    restart: always
    image: eclipse-mosquitto
    container_name: mosquitto
    ports:
      - 1883:1883
      - 9001:9001
    volumes:
      - /data/docker/home-assistant/mosquitto/config/:/mosquitto/config/
      - /data/docker/home-assistant/mosquitto/log/:/mosquitto/log/
      - /data/docker/home-assistant/mosquitto/data/:/mosquitto/data/
    networks:
      IoT:
        ipv4_address: 192.168.20.14
      proxy:
    labels:
      - traefik.enable=true
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"

# HomeAssistant
  homeassistant:
    container_name: home-assistant
    image: homeassistant/home-assistant:stable
    volumes:
      - /data/docker/home-assistant/config:/config
      - /data/docker/home-assistant/media:/media
    restart: always
    environment:
      - USER_UID=911
      - USER_GID=911
      - TZ=Europe/Berlin
    networks:
      IoT:
        ipv4_address: 192.168.20.10
      proxy:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"
      - traefik.enable=true
      - traefik.http.services.hass.loadbalancer.server.port=8123
      - traefik.http.routers.hass.rule=Host(`hass.berger-em.net`)
      - traefik.http.routers.hass.entrypoints=websecure
      - traefik.docker.network=proxy
    depends_on:
      - homegear
      - mariadb
      - mqtt

# ESPHome
  esphome:
    container_name: esphome
    image: esphome/esphome:latest
    ports:
      - 6052:6052
    volumes:
      - /data/docker/home-assistant/esphome:/config
    restart: unless-stopped
    networks:
      IoT:
        ipv4_address: 192.168.20.16
      proxy:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"
      - traefik.enable=true
      - traefik.http.services.esphome.loadbalancer.server.port=6052
      - traefik.http.routers.esphome.rule=Host(`esphome.berger-em.net`)
      - traefik.http.routers.esphome.entrypoints=websecure
      - traefik.docker.network=proxy

# Signal REST
  signal-cli-rest-api:
    image: bbernhard/signal-cli-rest-api:latest
    container_name: signal-rest
    environment:
      - MODE=json-rpc #supported modes: json-rpc, native, normal. json-prc is recommended for speed
    ports:
      - "8080:8080" # map docker port 8080 to host port 8080.
    volumes:
      - "/data/docker/home-assistant/signal:/home/.local/share/signal-cli" # map "signal-cli-config" folder on host system into docker container. the folder contains the password and cryptographic keys when a new number is registered
    restart: unless-stopped
    networks:
      IoT:
        ipv4_address: 192.168.20.15
      proxy:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
      - "com.centurylinklabs.watchtower.monitor-only=true"
      - traefik.enable=true
      - traefik.http.services.signal.loadbalancer.server.port=8080
      - traefik.http.routers.signal.rule=Host(`signal.berger-em.net`)
      - traefik.http.routers.signal.entrypoints=websecure
      - traefik.docker.network=proxy

networks:
  # Connet to the IoT VLAN
  IoT:
    external: true
  proxy:
    external: true
{{< /code >}}
**Note:** Treafik has its own compose file

## How it plays together
The main application in my setup is Home Assistant. Every container and many other devices in my house are connected in some way to Home Assistant. For all this "Smart-Devices" I defined a separate VLAN (Virtual Network) which is accessible via LAN and WLAN and has its own defined IP range (192.168.20.0/24). The docker containers got defined IP addresses in this network via [macvlan](https://docs.docker.com/network/macvlan/). To access the network from outside there are several firewall rules defined on my UDM (Unify Dream Machine).

For each of those containers I could write its own tutorial and writeup on how to set it up - and maybe I will in the future - but for now I will give you a short description for each one.

### Home Assistant
As mentioned above, this application is more or less the brain for our house. It not only controls lights, music, heating, etc.. it also uses many automations to make our life easier. For example there is one that will tell you when our washing machine is finished or one to alarm us when the rooftop windows are open and it starts to rain.

### MariaBD
This is the Database container for Home Assistant to get better and faster history data.

### Zigbee2MQTT
The name will tell you already, this container connects zigbee devices like lights to MQTT.

### Mosquitto
My MQTT broker -> Connects Zigbee2MQTT and Home Assistant together

### ESPHome
This is a project from the same group which is developing Home Assistant and a really great and easy way to integrate custom sensors and tool based on micro controllers into you Smart Home

### Homegear
I use Homegear to connect my old Homematic and Intertechno devices to Home Assistant. This tool will be the next part that I have to refactor in my system. The tool is to bulky and slow in comparison to all the other applications.

### Signal-Cli-Rest-Api
I don´t think you need any additional information about this container - it basically allows Home Assistant to send Signal messages. :)

### Traefik
I use Traefik for all my internal routing of http and https traffic. This container is not only used by my Smart Home system.

## Summary
Over all it requires more time to maintain, update and refactor than I thought while starting this project. But the system made its way into my family's daily business - "Alexa turn off the kitchen light" - and even my wife loves the easy and conformable way to control smart devices with this system. 