---
layout: post
title: "Setting up a Matrix Dendrite Homeserver on AWS Lightsail"
date: 2024-03-08
categories: linux servers
---

This post serves as a reference for a project. I will go over the steps taken to setup a self-hosted [Matrix](https://matrix.org/) homeserver, running Dendrite on AWS Lightsail with minimal resources.

## The Matrix protocol

Matrix is:

> An open network for secure, decentralised communication

Essentailly, anyone can run their own homeserver, which *federates* with other homeservers in a decentralised manner - similar to P2P or blockchain networks - creating a [fediverse](https://www.theverge.com/24063290/fediverse-explained-activitypub-social-media-open-protocol). One well-known social media use case of this technology is Mastodon. I decided on this project as I believe it has a great potential future, decentralising cloud native architecture increases reliability and resilience. As an open network, you are in control of your own privacy and security - not the case with a centralised company that stores millions of user data on servers under their control.

## High level overview

![](public/blog/matrix-aws.png)

Components:

- Dendrite second gen homeserver
	- small memory footprint - better for deploying on virtual instances
	- scalable - can run on multiple machines/instances
	- sqlite built in - easy deployment
		- can be more efficient with postgresql
- 1 Amazon Lightsail instance - an easy VPS service, less scalable than EC2 but great for a small project
	- $5 plan (3 months free, perfect for the duration of the course)
		- 1 GB memory (minimum requirement for the server installation)
		- 2 vCPUs
		- 40 GB SSD storage
		- 1 TB data transfer

## Getting a new domain name

Straightforward, bought a new domain name for this project. Configuring DNS records will come after creating an instance, as I describe in the next step.

## AWS Lightsail instance

I chose to use Lightsail over EC2 after comparing the pros and cons [here](https://aws.amazon.com/free/compute/lightsail-vs-ec2/). We're not hosting any enterprise grade applications here, and it's really a dev/test environment for the duration of the course, so it's the better option.

I select Debian for the OS with no additional applications, which allows me to configure the server from scratch. Steps taken:

- generate keypair for SSH access
- add a firewall rule for HTTPS traffic over port 443
- SSH into the server
- `sudo apt update && sudo apt upgrade`
- disable root login via SSH

## Configuring DNS records

All that's needed in the DNS configuration is an A record that points to the public IP of my instance. I went with matrix for the hostname/subdomain.

## Installation planning

Dependencies:

- Go 1.20+ (apt repo version is 1.15)
- reverse proxy (nginx)
- I will use the Docker container for a monolith installation for test purposes

## Docker preparation

First need to setup apt to get the correct Docker packages:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

After installing Docker and running a hello world container test to make sure it's all good, I need pull the image and generate a private key for Dendrite:

```
mkdir -p ./config
sudo docker run --rm --entrypoint="/usr/bin/generate-keys" \
  -v $(pwd)/config:/mnt \
  matrixdotorg/dendrite-monolith:latest \
  -private-key /mnt/matrix_key.pem

```

Next generate a config:

```
mkdir -p ./config
sudo docker run --rm --entrypoint="/bin/sh" \
  -v $(pwd)/config:/mnt \
  matrixdotorg/dendrite-monolith:latest \
  -c "/usr/bin/generate-config \
    -dir /var/dendrite/ \
    -db postgres://dendrite:itsasecret@postgres/dendrite?sslmode=disable \
    -server YourDomainHere > /mnt/dendrite.yaml"

```

In the config I set disable_federation to true, for test purposes. I left disable_registration set to true, because enabling registration without a secondary verification method such as captcha significantly increases risk of spam and abuse on the server. At this stage I won't be enabling a captcha.

After this I copy the docker-compose file and give it a test run. All seems good, time to setup nginx and TLS with [certbot](https://certbot.eff.org/).

```
sudo apt install -y nginx snapd
sudo snap install --classic certbot

# ensure certbot command can be run
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

After packages are installed, I configure nginx. Firstly create the site config file in sites-available:

```
cd /etc/nginx/sites-available
ln -s /etc/nginx/sites-available/<sitename> /etc/nginx/sites-enabled/<sitename>
```

Then use certbot --nginx to generate SSL certs and edit the nginx config file to serve over HTTPS.

## Testing deployment

Using the [federation tester](https://federationtester.matrix.org) I did a check on my domain name, which was a success.