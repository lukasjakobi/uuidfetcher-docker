# UUIDFetcher

![](https://img.shields.io/badge/-docker-informational)
![](https://img.shields.io/badge/version-v1.0.0-important)

UUIDFetcher is a ready to use docker stack including a few very handy software
tools, namely:

- Homeassistant
- Zigbee2MQTT
- Mosquitto
- Node-RED
- Portainer
- Pi-Hole
- Postgres
- Uptime Kuma

## Prerequisites

- Running installation of docker
- Zigbee USB Dongle (e.g. Sonoff Zigbee 3.0 Plus)
- Hardware to run docker stack on (e.g. Raspberry Pi)

## Configuration

This repository provides a basic home automation setup with a lot of things preconfigured,
however you will have to do some configuration on your own.

### 1. Zigbee2MQTT and Mosquitto

You need to create a username and password for your Mosquitto installation.
For this, run 

```shell
mosquitto_passwd -c ./mosquitto/config/mosquitto.conf <your_username>
```

Please note, that any existing files will be overridden by this command, because of the `-c` modifier.

Next, add this username and password combo into the `zigbee2mqtt/data/configuration.yaml` file under `mqtt`-> `user` and `password`.

Last, edit your USB Dongle ID. There are countless tutorials on how to find your USB Dongle IP, like [this one](https://example.com).
Insert this ID into the zigbee2mqtt node in the `docker-compose.yaml` file.

```
  ...
  zigbee2mqtt:
    image: koenkk/zigbee2mqtt
    container_name: zigbee2mqtt
    restart: unless-stopped
    volumes:
      - ./zigbee2mqtt/data:/app/data
      - /run/udev:/run/udev:ro
    ports:
      # frontend port
      - 8080:8080
    environment:
      - TZ=Europe/Berlin
    devices:
      # sonoff zigbee usb dongle
      - <YOUR_ZIGBEE_USB_DONGLE_ID>:/dev/ttyACM0 <--- INSERT HERE
  ...
```

### 2. Pi-Hole

The configuration for the Pi-Hole Container assumes you would like to define a custom password for the Pi-Hole UI.
Pi-Hole will by default create a random password during first startup, but this requires you to look at the container logs to
find this password.

To define your custom password head to `docker-compose.yaml` and edit the environment variable `WEBPASSWORD` in the pihole node.
This node looks something like

```
  ...
  pihole:
    container_name: pihole
    image: pihole/pihole
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80/tcp"
    environment:
      TZ: 'Europe/Berlin'
      WEBPASSWORD: 'CHANGE_ME'    <---- EDIT THIS LINE
    volumes:
      - './pihole:/etc/pihole'
      - './pihole/dnsmasq/dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
  ...
```

This means the default password for Pi-Hole will be `CHANGE_ME`.

### 3. Edit Time Zones (optional)

You can edit the timezones for Homeassistant and the other containers in the `docker-compose.yaml` file, if you wish to.


## Installation

```shell
docker compose up -d
```

Legacy docker installation may require you to use `docker-compose up -d` instead

### Home Assistant

This Home Assistant install is provided with a default `configuration.yaml` file.
The Service will be reachable under `<local_ip>:8123`, which is the default for Home Assistant.

As this is the docker version of Home Assistant, you will not be able to install Add-Ons.
That's why I provided the most useful plugins out of the box.