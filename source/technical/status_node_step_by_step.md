---
id: status_node_step_by_step
title: Status Node Step-by-Step
description: Run a Status Node in less than 30 minutes
---

## Status Node Step-by-Step

You can run a Status Node in less than 30 minutes. What you have to do is to enter a couple of commands to run a docker image. 

If you have trouble deploying a Linux machine to run a docker iamge, you can find how to deploy an AWS EC2 cloud instance [here](./others/deploy_ec2_instance.html), or set your own with other cloud providers based on the guideline. This guide assumes you've correctly opened the TCP and UDP port `30303` on your device.

### 1. Install and enable docker

1. `sudo apt-get update`
2. `sudo apt install docker.io`
3. `sudo systemctl start docker`
4. `sudo systemctl enable docker`
5. `docker --version`
6. `sudo usermod -aG docker $USER`
7. `newgrp docker`

<img src="./status_node_step_by_step/docker.png" alt="docker" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

### 2. Install and enable docker-compose

1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `docker-compose --version`

<img src="./status_node_step_by_step/docker-compose.png" alt="docker-compose" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

### 3. Install make, jq, and qrencode

`sudo apt install make jq qrencode`

### 4. Clone Status-go

1. `git clone https://github.com/status-im/status-go.git`
2. `cd status-go/`

<img src="./status_node_step_by_step/clone-status-go.png" alt="clone-status-go" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

### 5. Run a Status Node

`make run-mailserver-docker`

<img src="./status_node_step_by_step/run-a-status-node.png" alt="run-a-status-node" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

### 6. Health Check

1. `export DATA='{"jsonrpc":"2.0","method":"admin_peers","params":[],"id":1}'`
2. `curl -s -H 'content-type: application/json' -d $DATA localhost:8545 | jq .`

<br/>

If there are peers connected,
`curl -s -H 'content-type: application/json' -d "$DATA" localhost:8545 | jq -r '.result[].network.remoteAddress'`
will return IP addresses

<img src="./status_node_step_by_step/health-check.png" alt="health-check" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

### 7. Check Waku envelopes

- `curl -s localhost:9090/metrics | grep '^waku_envelopes_received_total'`

<img src="./status_node_step_by_step/waku-envelopes.png" alt="waku-envelopes" style="max-width:1000px;width:100%;margin:20px auto;padding:10px 0 10px 0;"/>

## Troubleshooting

#### Where is config.json?

The config file is located at `/var/tmp/status-go-mail/`.

#### How can I check the clusterConfig?

Run `jq '.ClusterConfig.BootNodes' /var/tmp/status-go-mail/config.json`.

#### How can I check the Status Node version?

Run `docker ps` and check out the image name.

#### How can update the docker image?

- Drop old docker container: `make clean-mailserver-docker`
- Clean up images: `docker image prune -f -a`
- Run again: `make run-mailserver-docker`

#### How can check peer information?

Run `curl -s localhost:9090/metrics | grep '^p2p_peers’`

#### No permissions to run `docker`

Enable your user to properly use the docker commands without using sudo for every command.

1. `sudo usermod -aG docker $USER`
2. `newgrp docker`

#### No `docker-compose` in your $PATH. Please install it.

Install [Docker Compose](https://docs.docker.com/compose/).

1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `docker-compose --version`

#### No `jq` in your $PATH. Please install it.

Install [jq](https://stedolan.github.io/jq/) by running `sudo apt-get install jq`.
