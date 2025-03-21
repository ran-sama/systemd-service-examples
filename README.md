# systemd-service-examples

A crash course for systemd and services, how to create and control them. Very basic usage with links for further reading.

## Why do we want to have a service?

It can be beneficially not to start a server with a CRON tab, because a service can be more easily restarted and it can restart itself after failure. The reasons can be as trivial as restarting a server after renewing your HTTPS certificates, so it may load the new ones.

## Creating our first unit file

A unit file is just plain text defining a service, a mount point or even much more components of your OS.

```
sudo systemctl edit --force --full fox4.service
```
It can be created with a simple oneliner, do note the self explanatory flags. Since we want to run a simple server, we only want to bring it up once the network is ready. This one is sufficient to run my python3 HTTPS and TLS1.3 server.

```
[Unit]
Description=fox4-server
#Wants=network.target
After=network.target

[Service]
#Environment="OPENSSL_CONF=/home/ran/pyopenssl.cnf"
Type=idle
#Type=notify
Restart=on-failure
#ExecStartPre=/bin/sleep 10
ExecStart=/usr/bin/python3 /home/ran/net/server_PV.py
#WorkingDirectory=/home/pi
StandardOutput=null
StandardError=journal
User=ran
Group=ran

[Install]
WantedBy=multi-user.target
```

Afterwards you can enable your new service:

```
sudo systemctl enable fox4.service
```

and manually bring it up

```
sudo systemctl start fox4.service
```

After these steps it should be running forever. You can check on it like this, sudo may reveal more info than running just as user in some cases:
```
sudo systemctl status fox4.service
```
The syntax overall needs not much further explanation:
```
sudo systemctl restart fox4.service
sudo systemctl stop fox4.service
sudo systemctl disable fox4.service
sudo systemctl reload fox4.service
```
Further useful commands are:
```
sudo systemctl daemon-reload
sudo systemctl list-unit-files
sudo systemctl list-timers --all
sudo systemctl list-units --type=service
```

## Special cases like Mumble/Murmurd

I was unhappy with Mumble server not being a "real" systemd service. It was more akin to a large bash file. Worse, it would not react to me restarting it. How to fix this?

```
sudo systemctl stop mumble-server.service
sudo systemctl disable mumble-server.service
```

Now to write a new one, do note the very much required ```-fg``` flags for it to work.

```
sudo systemctl edit --force --full murmur-new.service
```

```
[Unit]
Description=murmurd-new
After=network.target
Wants=network-online.target

[Service]
#Environment="OPENSSL_CONF=/home/ran/pyopenssl.cnf"
Type=simple
Restart=on-failure
ExecStartPre=/bin/sleep 2
ExecStart=/usr/bin/mumble-server -ini /etc/mumble/mumble-server.ini -fg
#User=mumble-server
#Group=mumble-server

[Install]
WantedBy=multi-user.target
```
This server will obey us when we want to restart it after a cert renewal job!

```
sudo systemctl enable murmur-new.service
sudo systemctl start murmur-new.service
```

Services created this way will be present in this directory:

```
cd /etc/systemd/system
cd /lib/systemd/system
```

## Further reading:

https://manpages.debian.org/bullseye/systemd/systemctl.1.en.html  
https://manpages.debian.org/bullseye/systemd/systemd.service.5.en.html  


## License
Licensed under the WTFPL license.
