# Consul Example

This repo demonstrates the uses of Hashicorp's Consul.

1. Service Registry
2. Health Checks
3. Key-Value (K/V) store

### Intro

#### Starting the Consul agent:

```bash
consul agent
```

Starting in "Dev" mode:

```bash
consul agent -dev
```

#### Querying Agents (Members)

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


### Configuration Directory

```bash
sudo mkdir /etc/conf.d
```

### Cluster
Assumming you're running more than one node and therefore have multiple nodes forming a cluster.

#### Start the Server & Client agents on the nodes
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

#### Joining a cluster

To join a cluster, run as the agent of any node and connect to one of the nodes already on the cluster by specifying that nodes' IP address.

e.g.
```bash
consul join 172.20.20.11
```

#### Query the nodes
Agent from one node querying another Agent from a second node.

e.g.
```bash
dig @127.0.0.1 -p 8600 agent-two.node.consul
```

#### Leave a cluster
```bash
consul leave
```

### K/V Store

The Key/Value store.

### Ui Web Console

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