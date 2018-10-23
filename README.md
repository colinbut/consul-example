# Consul Example

This repo demonstrates the uses of Hashicorp's Consul.

1. Service Registry
2. Health Checks
3. Key-Value (K/V) store

## Table of Contents

* [Intro](#intro)
    * [Starting Consul Agent](#starting-consul-agent)
	* [Querying Consul Agents (Members)](#querying-consul-agent)
* [Configuration Directory](#configuration-directory)
* [Services](#services)
* [HealthChecks](#healthchecks)
* [Consul Connect](#consul-connect)
    * [Intentions](#intentions)  
* [Cluster](#cluster)
  * [Start the Server & Client](#start-server-client)
  * [Joining Cluster](#joining-cluster)
  * [Query the Nodes](#querying-nodes)
  * [Leave Cluster](#leave-cluster)
* [Key/Value (K/V) Store](#kv-store)
* [UI Web Console](#ui-web-console)

### <a name="intro"></a>Intro

#### <a name="starting-consul-agent"></a>Starting the Consul agent:

```bash
consul agent
```

Starting in "Dev" mode:

```bash
consul agent -dev
```

#### <a name="querying-consul-agent"></a>Querying Agents (Members)

From CLI:

```bash
consul members --detailed
```

via HTTP API:
```
curl localhost:8500/v1/catalog/nodes
```

default port of `8500` applies

or via DNS interface:

e.g.
```
dig @127.0.0.1 -p 8600 ip-192-168-20-127.eu-west-1.compute.internal
```

"Note that you have to make sure to point your DNS lookups to the Consul agent's DNS server which runs on port 8600 by default."


### <a name="configuration-directory"></a>Configuration Directory

A 'config' directory for Consul. The directory stores:

1. Service definition files
2. HealthChecks definitions
3. Sidecar Proxy definitions

This config directory is commonly stored under `/etc/conf.d`

Create the folder before starting up Consul.

e.g.
```bash
sudo mkdir /etc/conf.d
```

### <a name="services"></a>Services

You define 'services' in a "Service Definition" file which is of JSON format. See this repo for samples.

You store the definition files in the Configuration Directory.

If Consul is already started up after you placed the 'new' Definition files in the Configuration Directory then issue either a Consul reload or SIGNUP

e.g.
```bash
consul reload
```

__Queries__
e.g.
```bash
curl http://localhost:8500/v1/catalog/service/web
curl 'http://localhost:8500/v1/health/service/web?passing'
```

or via DNS:
```bash
dig @127.0.0.1 -p 8600 rails.web.service.consul SRV
```

### <a name="healthchecks"></a>HealthChecks
```bash
curl http://localhost:8500/v1/health/state/critical
```

or via DNS:
```bash
dig @127.0.0.1 -p 8600 web.service.consul
```

### <a name="consul-connect"></a>Consul Connect

[TBC]

```bash
consul connect proxy -sidecar-for socat
consul connect proxy -service web -upstream socat:9191
consul connect proxy -sidecar-for web
```

#### <a name="intentions"></a>Controlling access with Intentions

Interactions in Consul are a way to specify whether a service can or cannot communicate with another service.

__Deny__
```bash
consul intention create -deny web socat
```

__Allow it again__
```bash
consul intention delete web socat

```

### <a name="cluster"></a>Cluster
Assumming you're running more than one node and therefore have multiple nodes forming a cluster.

#### <a name="start-server-client"></a>Start the Server & Client agents on the nodes
One agent acts as the 'Server' so you start up the agent as the Server.

```bash
consul agent -server -bootstrap-expect=1 \
-data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 \
-enable-script-checks=true -config-dir=/etc/consul.d
```

Another 'member' comes along to this cluster as the 'Client'
```bash
consul agent -data-dir=/tmp/consul -node=agent-two -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d
```

#### <a name="joining-cluster"></a>Joining a cluster

To join a cluster, run as the agent of any node and connect to one of the nodes already on the cluster by specifying that nodes' IP address.

e.g.
```bash
consul join 172.20.20.11
```

#### <a name="querying-nodes"></a>Query the nodes
Agent from one node querying another Agent from a second node.

e.g.
```bash
dig @127.0.0.1 -p 8600 agent-two.node.consul
```

#### <a name="leave-cluster"></a>Leave a cluster
```bash
consul leave
```

### <a name="kv-store"></a>K/V Store

The Key/Value store.

__Put__
```bash
consul kv put redis/config/minconns 1
consul kv put redis/config/maxconns 25
consul kv put -flags=42 redis/config/users/admin abcd1234
```

__Get one__
```bash
consul kv get redis/config/minconns
consul kv get -detailed redis/config/minconns
```

__Get All__
```bash
consul kv get -recurse
```

__Delete__
```bash
consul kv delete redis/config/minconns
consul kv delete -recurse redis
```

__Update__
```bash
consul kv put foo bar
consul kv put foo zip
```

__Atomic Update__
```bash
consul kv put -cas -modify-index=123 foo bar
```

### <a name="ui-web-console"></a>Ui Web Console

via the `-ui` flag on the CLI

```bash
consul agent -ui -config-dir=/etc/consul.d
```

and then you can visit the `/ui` path on the HTTP port of `8500`

e.g.
```
http://localhost:8500/ui/
```

The official Demo Consul UI is located here: `https://demo.consul.io/ui`