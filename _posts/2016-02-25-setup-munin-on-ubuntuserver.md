---
layout: post
title: Setting Up MUNIN on Ubuntu Server 14.04 with NGINX
image: http://munin-monitoring.org/site/munin.png
tags: [MUNIN, Linux]
comments: true
---

This post explains how to configure [munin](http://munin-monitoring.org/) on a single Ubuntu 14.04 server that uses [nginx](http://nginx.org/).

```bash
sudo apt-get update
sudo apt-get install nginx
```

Monitoring is supposed to save time by debugging and predicting / avoiding catastrophes. However, setting up munin on Ubuntu was a time-consuming trial-and-error process for me. The [official instructions](http://munin.readthedocs.org/en/latest/example/webserver/nginx.html) and various blog posts that cover this topic skip important steps, such as having monit's FastCGI processes start automatically at boot time. I have documented the setup that worked for me, hoping that others can reuse my work.


## Ubuntu packages
Run the following command to install the packages needed for munin.

```bash
sudo apt-get install munin munin-node spawn-fcgi libcgi-fast-perl
```

The following sections configure the munin packages.

## Munin configuration

Write the munin configuration below to `/etc/munin/munin-conf.d/90-fcgi`

```bash
graph_strategy cgi
html_strategy cgi
cgiurl_graph /munin/munin-cgi-graph
```

## NGINX configuration

Write the nginx configuration below to `/etc/nginx/sites-available/munin.conf`

```bash
server {
  #listen 443 ssl;
  listen 7000;
  charset utf-8;
  #server_name munin.your-domain.com;
  location ~ ^/munin/munin-cgi-graph/ {
    fastcgi_split_path_info ^(/munin/munin-cgi-graph)(.*);
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_pass unix:/var/run/munin/fastcgi-graph.sock;
    include fastcgi_params;
  }
  location /munin/static/ {
    alias /etc/munin/static/;
    expires modified +1w;
  }
  location /munin/ {
    fastcgi_split_path_info ^(/munin)(.*);
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_pass unix:/var/run/munin/fastcgi-html.sock;
    include fastcgi_params;
  }
  location / {
    rewrite ^/$ munin/ redirect; break;
  }
}
```

This configuration assumes that you have a `DNS` entry set aside for reaching the monit pages. I have separate DNS entries for all my applications, and they're all `CNAMEs` for the (same) machine that they're running on.
Once you're done tweaking the script, reload nginx.

```bash
sudo /etc/init.d/nginx reload
```

## FastCGI daemons
Write the script below to `/etc/init.d/munin-fcgi`

```bash
#!/bin/bash

### BEGIN INIT INFO
# Provides:          munin-fcgi
# Required-Start:    $remote_fs $syslog $network
# Required-Stop:     $remote_fs $syslog $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start munin FCGI processes at boot time
# Description:       Start the FCGI processes behind http://munin.*/
### END INIT INFO

graph_pidfile="/var/run/munin/fcgi_graph.pid"
# Ubuntu 12.10: /usr/lib/cgi-bin/munin-cgi-graph
graph_cgi="/usr/lib/munin/cgi/munin-cgi-graph"
html_pidfile="/var/run/munin/fcgi_html.pid"
# Ubuntu 12.10: /usr/lib/cgi-bin/munin-cgi-html
html_cgi="/usr/lib/munin/cgi/munin-cgi-html"

retval=0

. /lib/lsb/init-functions

start() {
  echo -n "Starting munin graph FastCGI: "
  start_daemon -p ${graph_pidfile} /usr/bin/spawn-fcgi -u munin -g munin \
      -s /var/run/munin/fastcgi-graph.sock -U www-data ${graph_cgi}
  echo
  echo -n "Starting munin html FastCGI: "
  start_daemon -p ${html_pidfile} /usr/bin/spawn-fcgi -u munin -g munin \
      -s /var/run/munin/fastcgi-html.sock -U www-data ${html_cgi}
  echo
  retval=$?
}
stop() {
  echo -n "Stopping munin graph FastCGI: "
  killproc -p ${graph_pidfile} ${graph_cgi} -QUIT
  echo
  echo -n "Stopping munin html FastCGI: "
  killproc -p ${html_pidfile} ${html_cgi} -QUIT
  echo
  retval=$?
}
case "$1" in
  start)
    start
  ;;
  stop)
    stop
  ;;
  restart)
    stop
    start
  ;;
  *)
    echo "Usage: munin-fcgi {start|stop|restart}"
    exit 1
  ;;
esac
exit $retval
```

Make the script executable, and fix some permissions while you're at it.

```bash
sudo chmod +x /etc/init.d/munin-fcgi
sudo chown munin /var/log/munin/munin-cgi-*
```

Now that the `init.d` script is in place, start it and have it run on every boot.

```bash
sudo /etc/init.d/munin-fcgi start
sudo update-rc.d munin-fcgi defaults
```

## Debugging
You should be able to point your browser to `http://[ip_address]:7000` and see many graphs. If that doesn't work out, the logs below should give you a clue as to what went wrong.

```bash
/var/log/nginx/error.log
/var/log/munin/munin-cgi-graph.log
/var/log/munin/munin-cgi-html.log
```

## Conclusion
I hope that you have found my post useful, and I hope that it helped you get your munin monitoring setup up and running in a manner of minutes.

> Thanks,