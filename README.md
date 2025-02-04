# IPv6 Neighbor Discovery Responder

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/yoursunny/ndpresponder/build)](https://github.com/yoursunny/ndpresponder/actions) [![GitHub code size](https://img.shields.io/github/languages/code-size/yoursunny/ndpresponder?style=flat&logo=GitHub)](https://github.com/yoursunny/ndpresponder)

**ndpresponder** is a Go program that listens for ICMPv6 neighbor solicitations on a network interface and responds with neighbor advertisements, as described in [RFC 4861](https://tools.ietf.org/html/rfc4861) - IPv6 Neighbor Discovery Protocol.

This program differs from [ndppd - NDP Proxy Daemon](https://github.com/DanielAdolfsson/ndppd) in that the source IPv6 address of neighbor advertisement is set to the same value as the target address in the neighbor solicitation.
This change enables **ndpresponder** to work in certain KVM virtual servers where NDP uses link-local addresses but *ebtables* drops outgoing packets from link-local addresses.
See my [blog post](https://yoursunny.com/t/2021/ndpresponder/) for more information.

## Installation

This program is written in Go.
It requires both Go compiler and C compiler.
You can compile and install this program with:

```bash
go install github.com/yoursunny/ndpresponder@latest
```

This program is also available as a Docker container:

```bash
docker build -t ndpresponder 'github.com/yoursunny/ndpresponder#main'
docker run -d --name ndpresponder --network host ndpresponder [arguments]
```

## Static Mode

The program can respond to neighbor solicitations for any address under one or more subnets.
It's recommended to keep the subnets as small as possible.

Sample command:

```bash
sudo ndpresponder -i eth0 -n 2001:db8:3988:486e:ff2f:add3:31e3:7b00/120
```

* `-i` flag specifies the network interface name.
* `-n` flag specifies the IPv6 subnet to respond to.
  You may repeat this flag to specify multiple subnets.

## Docker Network Mode

The program can respond to neighbor solicitations for assigned addresses in Docker networks.
When a container connects to a network, it attempts to inform the gateway router about the presence of a new address.

Sample command:

```bash
docker network create --ipv6 --subnet=172.26.0.0/16 \
  --subnet=2001:db8:1972:beb0:dce3:9c1a:d150::/112 ipv6exposed

docker run -d \
  --restart always --cpus 0.02 --memory 64M \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --cap-drop=ALL --cap-add=NET_RAW --cap-add=NET_ADMIN \
  --network host --name ndpresponder \
  ndpresponder -i eth0 -N ipv6exposed
```

* `-i` flag specifies the network interface name.
* `-N` flag specifies the Docker network name.
  You may repeat this flag to specify multiple networks.

## Other Options

You may change log level of this program by setting the `NDPRESPONDER_LOG` environment variable.
Acceptable values are `DEBUG`, `INFO`, `WARN`, `ERROR`, and `FATAL`.

Sample command:

```bash
sudo NDPRESPONDER_LOG=WARN ndpresponder [arguments]
docker run -e NDPRESPONDER_LOG=WARN [other arguments]
```
