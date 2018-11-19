## Info
This is the live working copy of my smart home. Im currently using a server running Ubuntu Server 18.04 LTS with Docker. 



## Containers

- Home Assistant
- Grafana: makes pretty graphs. Pulls data from InfluxDB 
- InfluxDB: database for storing history
- Organizr: Easy way to have everything organized. Does also include Nginx, which can be used as a reverse proxy. Deals with client certs and SSL.
- Mosquitto: MQTT broker
- Node-RED: Flow-based automation
- Portainer: GUI for managing containers and stuff related to Docker


## Hardware

- Aeotec Z-Stick Gen 5 Z-Wave
- Philips Hue Hub
- Sonos One, with Alexa support

## Installation

### Docker

There are many ways you can install docker.
https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1

I installed using the repository

```
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Check that you got the same fingerprint that is stated in the link above.

For x86_64 /amd64:


```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable
```


Now its time for installation

```
$ sudo apt-get update
$ sudo apt-get install docker-ce
$ sudo docker run hello-world
```

That last line should download an image an run it in a container. If everything is working, you should see some message.

Docker is good, but Im quite fond of Docker Compose, so lets install that as well.


```
$ sudo pip install docker-compose
```

Docker Compose makes it so much easier to handle several containers that are dependent on each other, You have one file that handles configuration of all your containers.



### Making those containers

Start by creating relevant directories

```$ sudo mkdir /opt
$ sudo chown username:usergroup /opt
$ cd /opt
$ mkdir homeassistant
$ mkdir grafana
$ mkdir influxdb
$ mkdir mosquitto
$ mkdir node-red
$ mkdir organizr
$ mkdir portainer
```

Next up is where the magic happens.

Copy docker-compose.yaml into /opt/

Make adjustments where needed. Most of it should be self explanatory, or do an online search. Notice that Im still using version 2.1, when 3.x is out. Had som problems with dependencies on v3, but feel free to tell me why I should switch. Right now everything is working as intended.


```$ cd /opt
$ docker-compose pull
```

This could take some time as there is quite a lot of data to be downloaded.


## Configuring the containers

### InfluxDB
For Home Assistant to be able to use InfluxDB, we need to preconfigure a couple of things. 


First we need to generate the default configuration file.
```$ docker run --rm influxdb influxdb config > /opt/influxdb/influxdb
```

 


### Home Assistant

### Mosquitto
Under construction

### Node-RED

### Organizr

### Portainer




