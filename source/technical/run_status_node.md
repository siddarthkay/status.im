---
id: run_status_node
title: Running Status Node
description: Status Node is an Ethereum client supporting the Status app.
---

## Why Run Status Node?

Currently, there are no crypto incentives for running Status Nodes. We are working hard to solve this problem. Our intent is to increase the size of the Waku network, improving how decentralized and safe our platform is.

### Privacy

Another reason is privacy. In the current setup, most nodes - both relay and historical ones - are running as part of Status infrastructure. This means that Status has a wide view of most of the network. While all traffic in Waku is encrypted, the metadata that could be gathered this way can leak some information. The best way to avoid that is to run your own node and configure it in the Status app.

### Community

Running your own node provides additional nodes for the Status community. We encourage anyone to publish the enode addresses of their nodes for others to use. We also recommend running them as a permanent service or a docker container, so that it keeps running after system restart or a runtime node error.

### Types of Nodes

* Relay Node - A regular Waku Node which relays messages between nodes, including mobile or desktop clients.
* History Node - Also known as a Mailserver, stores historical messages and delivers them when queried.
    - Requires additional disk space. Around 1 GB of free space would be a start for storing last 30 days.

## Running A Status Node

Status Node is a modified [go-ethereum](https://github.com/ethereum/go-ethereum) node called [status-go](https://github.com/status-im/status-go) running on a server and supporting the Status app. As we operate in a decentralized model, we need multiple peers scattered around the globe to provide a reliable service.

Status Nodes support relaying Waku messages, propagating them between nodes, and support storing them for devices that were offline when the message was sent.

### Requirements

A machine running Linux or MacOS is required. It is entirely possible to run a Status Node on a physical machine in a local network, but full functionality requires a public and static IP address via which the service can be accessed.

[Cloud service providers](https://en.wikipedia.org/wiki/Cloud_computing) are an alternative which provides a public and static IP automatically in most cases. Cloud service providers may also provide stronger guarantees of uptime, so that more envelopes can be collected for later retrieval.

1GB of RAM and 1 vCPU on a single instance is typically sufficient to run a Status Node reliably.

Minimum software requirements are `make` and `jq`. If you want to build `status-go` you will also need `golang`, version `1.13` or higher.
A nice-to-have is `qrencode` to display a QR Code with your `enode://` address.

Ex. for Ubuntu `20.04`:
```
sudo apt install make jq golang qrencode
```

### Ports

* `30303` TCP/UDP - [DevP2P](https://github.com/ethereum/devp2p) wire protocol port. Must __ALWAYS__ be public.
* `8545` TCP - [JSON RPC](https://github.com/ethereum/wiki/wiki/json-rpc) management port. Must __NEVER__ be public.
* `9090` TCP - [Prometheus](https://prometheus.io/docs/concepts/data_model/) metrics port. Should not be public.

## Quick Start

The quickest way to start a node is using our `Makefile` scripts. See [here](https://github.com/status-im/status-go/blob/develop/MAILSERVER.md) for details.

1. Clone the [status-go](https://github.com/status-im/status-go) repo.
2. Start the Node service using one of two options:

    A. Docker Container - Does not require building the node.
    ```sh
    make run-mailserver-docker
    ```
    For more details consult the [Docker](https://github.com/status-im/status-go/blob/develop/_assets/compose/mailserver) `README`.

    B. systemd service - Requires building the node.
    ```sh
    make run-mailserver-systemd
    ```
    For more details consult the [systemd](https://github.com/status-im/status-go/blob/develop/_assets/systemd/mailserver) `README`.

## Manual Approach

### Building

First, build a `statusd` binary:
```
mkdir ~/go/src/github.com/status-im
git clone https://github.com/status-im/status-go ~/go/src/github.com/status-im/status-go
cd ~/go/src/github.com/status-im/status-go
make statusgo
```
For more information visit [this page](./build_status/status_go.html).

### Running

You can check the available options using the `-h`/`--help` flags:
```bash
./build/bin/statusd -h
```
Note that the default settings will not let you run a full relay and history node.

### Configuration

The configuration is provided as a JSON file. Here is a basic config to run a Waku node that also stores historical messages:

`./config.json`
```json
{
    "AdvertiseAddr": "<YOUR_PUBLIC_IP>",
    "ListenAddr": "0.0.0.0:30303",
    "HTTPEnabled": true,
    "HTTPHost": "127.0.0.1",
    "HTTPPort": 8545,
    "APIModules": "eth,net,web3,admin,mailserver",
    "RegisterTopics": ["whispermail"],
    "WakuConfig": {
        "Enabled": true,
        "EnableMailServer": true,
        "DataDir": "/var/tmp/statusd/waku",
        "MailServerPassword": "status-offline-inbox"
    }
}
```

Provide it using the `-c` flag:
```bash
$ ./build/bin/statusd -c ./config.json
```

For more example config files, check out [this directory](https://github.com/status-im/status-go/tree/develop/config/cli). For details on the config options, consult this [README](https://github.com/status-im/status-go/blob/develop/config/README.md).

You can also read the comments for the options in [this source file](https://github.com/status-im/status-go/blob/develop/params/config.go).

### Metrics

To enable Prometheus metrics, use the following flags when running `statusd`:
```sh
./build/bin/statusd -metrics -metrics-port=9090
```
Metrics will be exposed on the `9090` port:
```sh
 > curl -s localhost:9090/metrics | grep '^whisper_envelopes_received_total'
whisper_envelopes_received_total 123
```

### Healthcheck

To check if the service is running, use the JSON RPC administration API:
```sh
 $ export DATA='{"jsonrpc":"2.0","method":"admin_peers","params":[],"id":1}'
 $ curl -s -H 'content-type: application/json' -d "$DATA" localhost:8545 | jq -r '.result[].network.remoteAddress'
34.68.132.118:30305
134.209.136.123:30305
178.128.141.249:443
```

### Using Docker

Status provides a [Docker image](https://hub.docker.com/r/statusteam/status-go/) that is used for running nodes on our fleet as well as using the [Docker Compose setup](https://github.com/status-im/status-go/tree/develop/_assets/compose/mailserver) we provide.

Ex. to run a container yourself:
```bash
docker run --rm \
    -p 8545:8545 \
    -p 30303:30303 \
    -v $(pwd)/config.json:/config.json \
    statusteam/status-go:0.55.1 \
    -register \
    -log DEBUG \
    -c /config.json
```
