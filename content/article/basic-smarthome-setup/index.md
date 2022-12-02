---
title: "Basic SmartHome Setup"
date: 2022-12-02T15:38:10+01:00
draft: true

categories: []
tags: ["home assistnt", "zigbee2mqtt", "traefik", "homegear", "Signal-Cli-Rest-Api", "mqtt"]
toc: true
---

In this article I will explain the basic setup for our/my SmartHome system. In the future there may be some additional articles for a more detailed view on some components.  

## Components

My SmartHome is based on docker containers running the following applications:
- [Home Assistant](https://home-assistant.io)
- [MariaDB](https://mariadb.org/)
- [Zigbee2MQTT](https://www.zigbee2mqtt.io/)
- [Homegear](https://homegear.eu/) (For Homematic and Intertechno)
- [Mosquitto](https://mosquitto.org/) (MQTT broker)
- [Signal-Cli-Rest-Api](https://github.com/bbernhard/signal-cli-rest-api)
- [Traefik](https://traefik.io/) (Not only for my SmartHome in use)

All my containers are started and maintained with docker-compose.

```yaml
docker-compose file here ;)
```

## How it plays together
The main application in my setup is Home Assistant. Every container and many other devices in my house are connected in some way to Home Assistant.