---
title: "Node Red"
date: 2021-11-28
tags:
- iot
- blockchain
- node-red
---

[Official Website](https://node-red.org/)

[Playlist](https://youtube.com/playlist?list=PLcjWRSA2O5d2tI7pzamHDCkaOuir_oVYq)

a visual editor to build workflow and ui application to interact with IoT

* wire the IoTs and user (with web dashboard)
* read data from IoT
* write data to IoT (control it, on and off etc)
* build a flow among IoTs and users for automation
* build dashboard UI

all can be done by drag and drop

can be used with blockchain in the supply chain...

node red pulls the data from sensors and submits to blockchain automatically

# Use cases

## Temperature monitor dashboard

* listen to any data sent from the thermometer
* show the temperature on the dashboard

## Light switcher

* send switch on and off signal to the light bulb through dashboard

## Automation (like IFFF and Apple Shortcut but in the IoT version)

* monitoring the environment with sensors (like humidity and temperature)
* trigger other IoT (like turn on the washing machine)


## Water Utility (SCADA)

https://www.youtube.com/watch?v=FCfmWnxQkoc

* tanks and bump
* bump water from the well to task 1 and then bump to tank 2
* alert when the tank reaches the critical level

# MQTT

lightweight network protocol for IoT

* [How to Get Started with MQTT](https://www.youtube.com/watch?v=tQmXWNd1pNk)
* [What is an MQTT Broker Clearly Explained](https://www.youtube.com/watch?v=WmKAWOVnwjE)

## Basic Concept

* IoTs subscribe to specific topic `workshop/switch` in MQTT Broker
* user publish `workshop/switch` to MQTT broker (via red node)
* IoTs receive the data from the MQTT broker and react to it (eg: turn on the switch...)

##  MQTT Broker

Open source MQTT broker: [Eclipse Mosquitto](https://mosquitto.org/)

### bidirectional protocol 

subscribe and publish at the same time in the same IoTs

### TLS

### Basic authentication

### CERT

### failover / backup

### retained message

When adding a new device, the value is empty as no subscribed topic is sending data. this feature will fill it with the last record

### persistent session

when IoT disconnected to the broker

* no need to resubscribe the topic
* automatically connect back to the broker with network connection back

### Client status

IoT sends birth death discount message when connected to broker

* When the connection is established, the broker sends a birth message to a set of topic's subscribers
* There is a keep-alive timer at the broker. Broker will ping the device periodically to ensure the device is alive
* If the keep -alive timer is timeout, the broker will send a death message to the topic's subscribers 

Also, red-node can ping the IoTs to check their status. publish the status of IoT topic when it is connected or disconnected or unexpected disconnected

MQTT.fx = software to publish / subscribe to the broker

# with PI

https://www.youtube.com/watch?v=WxUTYzxIDns

* mentions MQTT broker but does not mention how to setup
* node-red is installed on the pi itself, and sends a message to the pin directly to control the light bulb

## video does not tell 

* how to setup the MQTT (send data to a specific topic to the broker) 
* how to receive data from the device without MQTT (he just shows how to control the light bulb)

# MQTT client setup

### PI

https://www.youtube.com/watch?v=WxUTYzxIDns

* install MQTT client
* connect to the MQTT server
* publish all events to the MQTT service with a specific topic ([need to write code](https://www.youtube.com/watch?v=Pb3FLznsdwI))

### Device with preinstalled software

https://www.youtube.com/watch?v=KqHVkUiPRzc

This is not using node-red but the concept is same

* just enable MQTT function

## How to capture MQTT data from device at node-red

* listen to all topics from MQTT and capture it to debug node
* trigger the event to send out (eg: clicking the wifi button) / wait for the device to publish a message to the topic (thermometer)
* find out the payload and topic name

# Enterpise Case study

* [How to Build Massive IoT with Klika Tech, Wirepas and AWS](https://www.youtube.com/watch?v=GVqDQXi-Ls0)
* [Building an End-to-End Industrial IoT (IIoT) Solution with AWS IoT - AWS Online Tech Talks](https://www.youtube.com/watch?v=arpPt40jRUw&list=WL&index=9)

## Use case
* tracking the temperature, humidity, vibration in shipping
* pro-logistics: open the container door, turn on and turn off the light..
* control the lighting
* track which the conference room is used
* locate things in office / warehouse

## Flow

* IoTs
* gateways
* MQTT Broker (aggregate all data)
  * user application
  * [wirepas network tools](https://www.wirepas.com/)
  * backend

* wirepas FW (RuuviTag)
* wirepas snap
* (AWS) wirepas service (WNT, WPE) (Dashboard) / Greengrass snap (Like Red-Node)

## AWS IoT SiteWise & AWS IoT Greengrass

(Depends on the device manufactory)
* OnLogic
* Moxa

[greengrass](https://aws.amazon.com/greengrass)

access point for the IoTs to send data to AWS

## AWS IoT Message Broker
 
AWS version of MQTT

## AWS IoT Device Management 

 IoT version of System Manager 

* bulk registration
* Patching
* Monitor the health / Search
* Integration with analytics tools

## Wireless mesh network with asset tracking on site

* building gateways on site to let the IoTs can send data to cloud (SiteWise)
* the message can be repeated (extended) once they cover each other (that is why it is called a mesh network)
* 700000 devices in one mesh network cover Oslo
* ROI -> look at the maintanese and analytics and delivery

## RuuviTag

* Pressure
* Temperature
* Humidity
* Accelerometer
* Location

* I2C / SPI

## Steps

* Build SiteWise Assets / Model
* Build SiteWise Gateway (connect with KEPServerEX)
* Portal -> SiteWise Montior -> Create Projects
* IoT Core set a rule (How to process the data) to send data to analytics 
* in analaytics there is a channel created -> use pipeline to pass the raw data to lamda and process the raw data -> store the processed data to data store -> create (SQL / Container custom code) in data sets and consumer in IoT Events (Detector model)
