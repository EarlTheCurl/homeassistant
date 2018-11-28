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
DISCLAIMER: I always recommend to password protect or use other forms of authentication when exposing ports. The following "guide" wont always show you how, even though I personally have done it. Be careful and understand what you are doing in each step!


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

While in the same directory as `docker-compose.yaml` (will explain later) you can start containers using just
```
$ docker-compose up -d homeassistant
```


### Making those containers

Start by creating relevant directories

```
$ sudo mkdir /opt
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
Change username and group to correct values. We want your local user to be owner of these folders.

Next up is where the magic happens.

Download https://gist.github.com/EarlTheCurl/6ccba652616ec9cac3c2538b838ab06e and copy `docker-compose.yaml` into `/opt/`

Make adjustments where needed. Most of it should be self explanatory, or do an online search. Notice that Im still using version 2.1, when 3.x is out. Had som problems with dependencies on v3, but feel free to tell me why I should switch. Right now everything is working as intended.
```
$ cd /opt
$ docker-compose pull
```

This could take some time as there is quite a lot of data to be downloaded.


## Configuring the containers


### InfluxDB
For Home Assistant to be able to use InfluxDB, we need to preconfigure a couple of things. 


First we need to generate the default configuration file.
```
$ docker run --rm influxdb influxdb config > /opt/influxdb/influxdb
```

Then start the container using

```
$ docker-compose run -d influxdb
```

Using the Influx client, lets create a database
```
docker exec -it influxdb influx

> CREATE DATABASE home_assistant
> exit
```

Last step would be to connect Home Assistant to InfluxDB. Add the following lines to your `configuration.yaml`. 

```
influxdb:
  host: 127.0.0.1
  port: 8086
  database: home_assistant
  default_measurement: state
  include:
    entities:
      # add entities here
  exclude:
    domains:
      # add entities here
      - group
```

I recommend to password protect your database! I might add a howto, but not now. Google it!

Useful links:
- [InfluxDB Docker](https://hub.docker.com/_/influxdb/)
- [InfluxData](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/)


### Mosquitto
Add the following lines to 
`/opt/mosquitto/mosquitto.conf`

```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

allow_anonymous true

port 1883
```

Useful links:
- [Mosquitto config](https://mosquitto.org/man/mosquitto-conf-5.html)
- [Mosquitto Docker](https://hub.docker.com/_/eclipse-mosquitto/)


### Home Assistant
Not sure if there is much to be explained here.

Use the following command to start this conatiner
```
$ docker-compose run -d homeassistant
```

Note how its dependent on both Mosquitto and InfluxDB as stated in `docker-compose.yaml`

```
    depends_on:
      influxdb:
        condition: service_healthy
      mosquitto:
        condition: service_started
```

Which means that if either of these containers is not working, HA wont either. You could comment out these dependencies if you dont need them.



### Node-RED
```
$ docker-compose up -d node-red
```
Node-RED should be accessible from `http://YOUR.SERVER.IP.ADDRESS:1880`

Lets secure it.

First we need to generate a password hash as we dont like saving passwords in plain text. Open a Node-RED container shell:
```
$ docker exec -it node-red /bin/bash
```

Change out YOURPASSWORD to a superb password and run the command while in the shell.

```
node -e "console.log(require('bcryptjs').hashSync(process.argv[1], 8));" YOURPASSWORD
###GENERATED_HASH###
exit
```

Edit the following section of  `/opt/node-red/settings.js` with your new hash:
```
    // Securing Node-RED
    // -----------------
    // To password protect the Node-RED editor and admin API, the following
    // property can be used. See http://nodered.org/docs/security.html for details.
    adminAuth: {
        type: "credentials",
        users: [{
            username: "admin",
            password: "###GENERATED_HASH###",
            permissions: "*"
        }]
    },
```

Restart Node-RED
```
$ docker restart node-red
```

Connect Node-RED to Home Assitant: //Under construction




### Organizr
Under constructiom



### Portainer
Start container using
```
$ docker-compose up -d portainer
```

Should now be accessible from `http://YOUR.SERVER.IP.ADDRESS:9000`




## Trobuleshooting
Some bumps in the road that I experienced...

#### Apt install does not work
I learned that Ubuntu 18.04 Server does not have universe/multiverse/backports enabled by default.

```
$ nano /etc/apt/sources.list
```

Then add "universe" after each entry. This allows you to install packages such as `python-pip`. It should look like this.
```
deb http://archive.ubuntu.com/ubuntu bionic main universe
deb http://archive.ubuntu.com/ubuntu bionic-security main universe 
deb http://archive.ubuntu.com/ubuntu bionic-updates main universe
```
Save, exit and try again.

#### Grafana container wont start
Try manually creating an empty config file:
``` 
$ sudo nano /opt/grafana/grafana.ini
```


### Mosquitto: 0.0.0.0:1883:bind: address already in use
Im not sure if there is any better ways to do fix this (most likely is), but I just removed the port binding from the docker-compose file.

By that I mean:

```
  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto
    user: "1000:1000"
    ports:
      - 1883:*1883*
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /opt/mosquitto:/mosquitto/config:ro
      - /opt/mosquitto:/mosquitto/data
    restart: on-failure
```

Turns into

```
  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto
    user: "1000:1000"
    ports:
      - 1883
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /opt/mosquitto:/mosquitto/config:ro
      - /opt/mosquitto:/mosquitto/data
    restart: on-failure
```