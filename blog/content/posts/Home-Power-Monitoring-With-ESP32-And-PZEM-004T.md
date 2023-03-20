---
title: "Home Power Usage Monitoring With ESP32 and PZEM 004T-v3 Using Home Assistant"
summary: This post is about using the PZEM-004T energy meter module along with ESP32 running Tasmota firmware.
cover:
    image: "/assets/images/anticipatory-log-collection/cover.jpeg"
tags: ['operations', 'techops', 'aws', 'logging','cloudwatch']
categories:
- software
- hardware
- tasmota
- homeassistant
date: 2023-01-11T12:34:10+05:30
draft: true
---

Monitoring power consumption is an excellent first step when you're setting up a smart home or if you're stepping into smart homes.

This article explains the setup I used for setting up a real time power monitoring system for my home using the energy monitoring module PZEM-004T, ESP32 and Tasmota firmware publishing data to Home Assistant. This is one of the first steps I took towards converting my newly built "dumb home" to a "Smart Home".

This is a fairly advanced setup & certainly *dangerous* as it involves tinkering with the mains voltage.

**Hardware List**
- PZEM-004T v3.0 Energy Monitoring Module (I used one without case since I wanted to house it within a 3D printed case)
- ESP32 Module (I used a bare module from Espressif)
- 3D Printed DIN rail case to install the module within the power distribution box
- HiLink AC to 3.3v DC mini SMPS module
- 2 x Momentary push buttons
- 2 x 10k Resistor
- USB to UART adapter for programming the ESP32 module

First step is to setup the ESP32 module for flashing the Tasmota firmware - Tasmota is an open source firmware built as a replacement firmware for ESP8266/ESP32 based smart switches, relays & plugs. Its really powerful & the documentation is extremely comprehensive. I've been using a few modules running Tasmota since the last few years in my old home & it never failed. Tasmota has built in support for the PZEM-004T module & it can publish data via MQTT to the Home Assistant server.

Flashing Tasmota is farily simple, similar to flashing any other firmware on to ESP32. If you are using an ESP32 Dev Kit or a NodeMCU style board with built in USB to UART converter, its as simple as plugging in the device to your PC/Laptop and following the flash procedure. Follow the below schematic for flashing a bare ESP32 module:

![ESP32 Flashing](/assets/images/home-power-monitoring/esp32-minimal-flashing.png)

Once the connections are made and assuming you have all the necessary drivers installed, connect the USB to UART adapter to your PC/Mac and head over to https://tasmota.github.io/install/ using Google Chrome browser and click on "CONNECT". You'll see a popup with a list of devices along with your USB to UART adapter. Select your USB to UART adapter and click OK. Once connected, click on "Install Tasmota" and make sure to have the option "Erase Device" checked. Press and hold the BOOT button & then press the RESET button once, click on the final install button & you should see the messages "Erasing" and then installation progress - Do not navigate away or switch tabs while the installation is in progress.

It'll take about a minute or 2 to complete. Once completed, close Tasmota installer tab and head over to your phone or laptop and scan for new Wifi Access Points. You should see a new wifi access point named `tasmota-<alphanumeric code>`. Connect to the new access point and navigate to the IP address `192.168.4.1`. You should see the Tasmota web interface.

![Tasmota web UI](/assets/images/home-power-monitoring/tasmota-default.png)

Once the web UI is loaded, you're all set to configure your module.

Click on "Configuration" button & then click on "Configure WiFi", you should see something like the following:

![Tasmota config wifi](/assets/images/home-power-monitoring/tasmota-wifi-config.png)

Click on your Wifi network, enter your password and hit "Save". The module will restart and the access point `tasmota-<alphanumeric code>` will disappear. Once the device restarts, it'll connect to your wifi network, get the IP address of the module from your router and type it in your browser and hit go, you should see the Tasmota web UI - Tasmota initial config is now complete.

Remove the USB to UART adapter and the BOOT push button if you wish to - its no longer needed.

