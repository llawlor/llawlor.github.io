---
layout: post
comments: true
tags:
- Phusion Passenger
- Nginx
- systemd
- systemctl
title:  "Phusion Passenger & Nginx with Systemd Notes"
---

The recommended way to add Nginx to systemd is via the following from [https://www.nginx.com/resources/wiki/start/topics/examples/systemd/](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/){:target="_blank"}:

<pre>
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
</pre>

However, if you're using Phusion Passenger also and run the command `sudo passenger-status`, you may get the following error message:
<pre>
ERROR: Phusion Passenger doesn't seem to be running. If you are sure that it is running, then the causes of this problem could be:

1. You customized the instance registry directory using Apache's PassengerInstanceRegistryDir option, Nginx's passenger_instance_registry_dir option, or Phusion Passenger Standalone's --instance-registry-dir command line argument. If so, please set the environment variable PASSENGER_INSTANCE_REGISTRY_DIR to that directory and run passenger-status again.
2. The instance directory has been removed by an operating system background service. Please set a different instance registry directory using Apache's PassengerInstanceRegistryDir option, Nginx's passenger_instance_registry_dir option, or Phusion Passenger Standalone's --instance-registry-dir command line argument.
</pre>

This is because Phusion Passenger utilizes the `/tmp` directory, which has been modified via the systemd line above that reads `PrivateTmp=true`.  So if you want Phusion Passenger to run without any problems, you'll want to change `PrivateTmp=true` to `PrivateTmp=false`.

The correct systemd code for using Phusion Passenger with Nginx (and the default /opt/nginx path from the Phusion Passenger install) is:

<pre>
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/nginx/logs/nginx.pid
ExecStartPre=/opt/nginx/sbin/nginx -t
ExecStart=/opt/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=false

[Install]
WantedBy=multi-user.target
</pre>
