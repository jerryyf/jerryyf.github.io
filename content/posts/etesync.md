---
title: "Self-hosting EteSync"
author: jerryyf
date: 2023-06-24
toc: true
tags: ["self-hosting", "server", "privacy"]
---

[EteSync](https://www.etesync.com/) is an open source, end-to-end encrypted sync solution for contacts, calendars, tasks and notes. They offer their own SaaS, on a subscription-based model. [Etebase](https://github.com/etesync/server), their open source backend, can be used as a library for developing secure E2EE applications, or to host your own Etebase instance. In this post I cover my setup process and experience in hosting my own server (on a VPS).

<!--more-->

## Getting started

To start off, I grabbed a domain name and a cloud provider to host on. If you prefer full control over your own on-premise server consider building a home lab. This involves dedicated hardware, configuring router settings (you may have a dynamic IP address with your ISP, and/or need to configure port forwarding in order to access your server over the internet), or even outright replacing it. To save myself the hassle I just use a VPS.

For testing purposes however, the application can be served locally.

### Server setup

I went with an Ubuntu server as it's easy to setup and secure. Some key security steps I take on a new VPS:

#### Change the root account password

This can be done through the hosting provider's management console.

#### Full system upgrade

    $ sudo apt update && sudo apt upgrade

#### Create a standard user

This allows SSH access as a non-priveleged user rather than root user.

    $ adduser <your-user>
    $ sudo usermod -aG sudo <your-user>

#### Disable root access via SSH

Next we want to disable root access via SSH altogether for security purposes.

    $ sudo nano /etc/ssh/sshd_config

Modify this line:

    PermitRootLogin no

Restart the SSH daemon.
    
    $ systemctl restart sshd

## Etebase installation

SSH into the server. Make sure the Python version is 3.7 or newer - `python -V` to print the version.

Install `virtualenv` for Python 3:

- Arch Linux: `pacman -S python-virtualenv`
- Debian/Ubuntu: `apt-get install python3-virtualenv`

Clone the git repo and set up dependencies in Python virtual environment:

```shell
git clone https://github.com/etesync/server.git etebase

cd etebase

# Set up the environment and deps
virtualenv -p python3 .venv  # If doesn't work, try: virtualenv3 .venv
source .venv/bin/activate

pip install -r requirements.txt
```

## Configuration

I used the given configuration file `etebase-server.ini.example` in the repo:

    $ cp etebase-server.ini.example etebase-server.ini

In this file:

- set `ALLOWED_HOSTS` to `*` for testing purposes. we can change it to our domain name, e.g. `etebase.example.com` later.
- edit `STATIC_ROOT` and `MEDIA_ROOT` to `./static` and `./media` respectively.

The config file should now look something like this:

```ini
[global]
secret_file = secret.txt
debug = false
;Set the paths where data will be stored at
static_root = ./static
media_root = ./media

;Advanced options, only uncomment if you know what you're doing:
;static_url = /static/
;media_url = /user-media/
;language_code = en-us
;time_zone = UTC
;redis_uri = redis://localhost:6379

[allowed_hosts]
allowed_host1 = *

[database]
engine = django.db.backends.sqlite3
name = db.sqlite3

[database-options]
; Add engine-specific options here, such as postgresql parameter key words

;[ldap]
;server = <The URL to your LDAP server>
;search_base = <Your search base>
;filter = <Your LDAP filter query. '%%s' will be substituted for the username>
; In case a cache TTL of 1 hour is too short for you, set `cache_ttl` to the preferred
; amount of hours a cache entry should be viewed as valid:
;cache_ttl = 5
;bind_dn = <Your LDAP "user" to bind as. Must be a bind user>
; Either specify the password directly, or provide a password file
;bind_pw = <The password to authenticate as your bind user>
;bind_pw_file = /path/to/the/file.txt
```

For now we will leave allowed hosts set to `*` for testing purposes. Make sure to change it to our domain name later.

## ASGI server

ASGI (Asynchronous Server Gateway Interface) is the successor to WSGI (Web Server Gateway Interface), in which your application is a callable by default. Etebase recommends using uvicorn for ASGI. To install it in the virtual environment, run:

    $ pip3 install uvicorn[standard]

## Testing the application

Let's run the application for the first time:

    $ ./manage.py migrate
    $ uvicorn etebase_server.asgi:application --port 8000 --host 0.0.0.0

We can access our remote server by browsing to either `0.0.0.0:8000` if testing locally, otherwise to your server's public IP address followed by the port number, i.e. `192.168.x.x:8000`. A page saying "It works!" should come up.

## Production deployment with nginx

So far, we have a functioning debug server. For deployment for access over the internet and to the outside world, a proper tech stack is required for security and reliability. Here I use the same stack as follows from the Etebase documentation:

    the web client <-> Nginx <-> the socket <-> uvicorn <-> fastapi/Django

## Installing nginx

Simply install `nginx` from package manager.

    $ sudo apt-get install nginx

At this point we should make make sure the nginx systemd service can be enabled and run without problems:

    $ sudo systemctl status nginx
    $ sudo systemctl enable nginx
    $ sudo systemctl start nginx

Test nginx with:

    $ nginx -t

Error log is located at `/var/log/nginx/error.log`.

## Setting up nginx

Firstly create Django's static files for nginx to access.

    $ ./manage.py collectstatic

In `/etc/nginx/sites-available` create a new configuration file for Etebase called `etebase_nginx.conf`:

```conf
# the upstream component nginx needs to connect to
upstream etebase {
    server unix:///tmp/etebase_server.sock; # for a file socket
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or domain name
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    location /static/ {
        alias /static/; # Project's static files
    }

    location / {
        proxy_pass http://etebase;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
    }
}
```

Remember to change `server_name`. In this configuration a [Unix file socket](https://www.howtogeek.com/devops/what-are-unix-sockets-and-how-do-they-work/) is used to expose the application rather than a web port socket. Next symlink this file to `/etc/nginx/sites-enabled/etebase_nginx.conf`:

    $ sudo ln -s /etc/nginx/sites-available/etebase_nginx.conf /etc/nginx/sites-enabled/etebase_nginx.conf

Now test and restart nginx and tell uvicorn to use the file socket:

    $ nginx -t
    $ systemctl restart nginx
    $ uvicorn etebase_server.asgi:application --uds /tmp/etebase_server.sock

At this point we can edit `etebase_server.ini` and change allowed hosts to our domain name.

## Starting uvicorn at boot

We use `systemd` to handle autostarting uvicorn by creating a unit file for it under `/etc/systemd/system/`.

```
[Unit]
Description=Execute the etebase server.

[Service]
WorkingDirectory=path/to/etebase
ExecStart=/path/to/etebase/.venv/bin/uvicorn etebase_server.asgi:application --uds /tmp/etebase_server.sock

[Install]
WantedBy=multi-user.target
```

As with nginx, enable and start this service.

    $ sudo systemctl enable etebase_server
    $ sudo systemctl start etebase_server

Check its status:

    $ sudo systemctl status etebase_server

## HTTPS setup

Setting up HTTPS is a breeze with [Certbot](https://certbot.eff.org/). 

### Certbot installation

First we need to install snapd. It should already be pre-installed on Ubuntu 18.04 and above.

Confirm snapd is up-to-date:

    $ sudo snap install core; sudo snap refresh core

Install Certbot:

    $ sudo snap install --classic certbot

To be able to run `certbot` from anywhere on the command line, symlink it to `/usr/bin/certbot`:

    $ sudo ln -s /snap/bin/certbot /usr/bin/certbot

Then simply run the command with the nginx (in my case) option:

    $ sudo certbot --nginx

Certbot will automatically edit the nginx configuration to serve over HTTPS with a TLS certificate. A cron job or systemd timer also automatically renews certficates before they expire - this can be tested with:

    $ sudo certbot renew --dry-run

The renewal command is stored in one of the following locations:

- `/etc/crontab/`
- `/etc/cron.*/*`
- `systemctl list-timers`

Confirm Certbot did its job by visiting `https://your-website.com`.
