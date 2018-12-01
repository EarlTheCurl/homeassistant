# Trobuleshooting
Some bumps in the road that I experienced...

## Apt install does not work
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

## Grafana container wont start
Try manually creating an empty config file:
``` 
$ sudo nano /opt/grafana/grafana.ini
```


## Mosquitto: 0.0.0.0:1883:bind: address already in use
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

## Can't install palettes to Node-RED
Run `sudo chmod -R 0777 node-red/` while in `/opt` directory. Changes permissions of that folder and sub-folders. I'm unfamiliar with different permissions, so could be more correct permissions to use than 0777.

