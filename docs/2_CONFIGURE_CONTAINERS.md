# Creating those containers

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
$ mkdir watchtower
$ mkdir logs
```
Change username and group to correct values. We want your local user to be owner of these folders.

Next up is where the magic happens.

Download my [docker-compose.yaml](https://gist.github.com/EarlTheCurl/6ccba652616ec9cac3c2538b838ab06e) and copy it into `/opt/`

Just recently updated from v2.1 to v3.7 of Docker Compose. I have not found any problems yet..

```
$ cd /opt
$ docker-compose pull
```

This could take some time as there is quite a lot of data to be downloaded.


## Configuring the containers

### Home Assistant
Not sure if there is much to be explained here.

Use the following command to start this conatiner
```
$ docker-compose run -d homeassistant
```

Note how its dependent on both Mosquitto and InfluxDB as stated in `docker-compose.yaml`. Since Im using Influx for storing data and Mosquitto as a broker for some home made sensors, I want to be sure that those containers are up and running.

```
    depends_on:
      - influxdb
      - mosquitto
```

Which means that if either of these containers is not working, HA wont either. You could comment out these dependencies if you dont need them.



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

I recommend to password protect your database! I might add a howto, but thats for the future. Google it!



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



#### Connection Node-RED to HA
//Under construction

I assume that you are familiar with Node-RED, so won't cover this in detail. But it goes something like this:

Install `"node-red-contrib-home-assistant"`

Get a Long-Lived Access Token (LLAT) from HA by clicking your profile button in the upper left corner of the HA interface. At the bottom, there is a option to create a LLAT. Create it and copy it.

Back in Node-RED, under the config tab, double click "Home Assistant". Fill in your token and `http://YOUR.SERVER.IP.ADDRESS:9000` into where it should.

Using Node-RED is a book by it self, so won't cover that at all. 



### Portainer
Start container using
```
$ docker-compose up -d portainer
```

Should now be accessible from `http://YOUR.SERVER.IP.ADDRESS:9000`


### Watchtower
//Under construction


### Organizr
See the [next step](docs/3_SECURE_CONNECTION.md)
