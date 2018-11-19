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

