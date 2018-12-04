#External access using Dynamic DNS

At this point, you should have a working, but local, system. To be able to access Home Assistant from outside your local network, there is a couple of things to be configured. 
There are several options available and if you are unfamiliar terms such as portforwarding, SSL and certificates, I would probaly advice you not to do this. Or at least be careful. You dont want to expose your unsecured devices to the internet. 

I also recommend taking at using a VPN.

## DuckDNS
Go to https://duckdns.org and create an account. Pick a unique subdomain, such as `uniquedomain.duckdns.org`. This will your way of accessing Home Assistant from everywhere.
One the same page you should have a token. Copy that.

While in `opt/` create a new directory
```
$ mkdir duckdns
$ cd duckdns
```

Create a new shell script file called `/opt/duckdns/duck.sh` and put this into that file. Be sure to change YOURDOMAIN and YOURTOKEN.
```
echo url="https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOURTOKEN&ip=" | curl -k -o /opt/duckdns/duck.log -K -
```

Now we have to make it executable, and add it to cron so that it runs every 5 minutes.

```
$ chmod u+x duck.sh
$ crontab -e
```

Put this at the bottom of the crontab

```
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

Save and exit ([CTRL] + [O], then  [CTRL] + [x])

Test the script:

```
$ ./duck.sh
$ cat duck.log
```

Last command should return OK. If it return KO, check thah token and domain are correct in cat `duck.sh`

## Port forwarding
Both port 80 and 443 should be port forwarded to the static IP of your server. 

##SSL using Certbot and Let's Encrypt
Im using a modified version of [this](https://community.home-assistant.io/t/set-up-encryption-using-lets-encrypt/42513)

```
$ mkdir certbot
$ cd certbot
$ wget https://dl.eff.org/certbot-auto
$ chmod a+x certbot-auto
$ ./certbot-auto certonly --standalone --preferred-challenges http-01 --email YOUR@EMAIL.XYZ -d YOURDOMAIN.duckdns.org
```

Lets create certificates for relevant subdomains:
```
$ ./certbot-auto certonly --standalone --preferred-challenges http-01 --email YOUR@EMAIL.XYZ -d ha.YOURDOMAIN.duckdns.org
$ ./certbot-auto certonly --standalone --preferred-challenges http-01 --email YOUR@EMAIL.XYZ -d nodered.YOURDOMAIN.duckdns.org
$ ./certbot-auto certonly --standalone --preferred-challenges http-01 --email YOUR@EMAIL.XYZ -d grafana.YOURDOMAIN.duckdns.org
```

Now we want to auto-renew those certificates using cron job. Run:

```
sudo crontab -e
```

and add the following line at the bottom. 
```
X Y * * * ~/certbot-auto renew >> /opt/logs/certbot-renew.log
```
`X`: minute
`Y`: hour

That will renew your certificates  `X` minutes past `Y` hour every day. Documentation recommends using a random `X`, to distribute the load on renewal servers (I presume). Could probably [randomize that](https://stackoverflow.com/questions/9049460/cron-jobs-and-random-times-within-given-hours), but thats too much of a hassle for me. 

*Note*: You might want to change `~/certbot-auto` to where you have saved it. 
Easy way to test your cron job is by [running](https://askubuntu.com/questions/230476/how-to-solve-permission-denied-when-using-sudo-with-redirection-in-bash/230482#230482)
```
sudo bash -c "~/certbot-auto renew >> /opt/logs/certbot-renew.log"
```

