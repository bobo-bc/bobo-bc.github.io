---
layout: post
title: "Swarm install"
date: 2025-09-28 18:30:00 +0800
categories: [swarm]

---

## Install Docker Swarm

On node1 with IP address 10.0.0.20 initialize the Docker Swarm.

```bash 
docker swarm init --advertise-addr 10.0.0.20
```

It should return us output similar to this:

```bash
Swarm initialized: current node (myjwx5z3m7kcrpli.......) is now a manager.
```

To add a worker to this swarm, run the following command:

```bash
docker swarm join --token SWMTKN-1-2gwqttpup0hllec6p6xkun8nmht4xu18g09vsxyjhlyqc........... 10.0.0.20:2377
```

To add a manager to this swarm, run 

```bash
docker swarm join-token manager
```
 and follow the instructions.


Since we are creating a high available cluster of three nodes we will not be adding workers.


## Join managers
In Docker Swarm, any node can be designated as a Master node, while all nodes are considered Worker nodes and highlights the flexibility and scalability of Docker Swarm.
Run the following on node1.

```bash
docker swarm join-token manager
```

Follow the instructions pasting into the command line on node2 and node3

## Check
On any of the manager node, run

```bash
docker node ls
```

## Network
Bug: The overlay ingress network that is automatically created was in the same range as the nodes if you are using 10.x.x.x network. If you are configured using 192.x.x.x. or 172.x.x.x then skip ahead.
Our Nodes have IPs from 10.0.0.x and the network was set
You can check that with commands:

```bash
docker network ls
```

| NETWORK ID   | NAME                  | DRIVER  | SCOPE |
|--------------|-----------------------|:-------:|:-----:|
| kpiocqticjlx | ingress               | overlay | swarm |
| k55h53e1e97d | portainer_agent_network | overlay | swarm |

using the Network ID of the ingress network

```bash
docker network inspect --format='{{json .IPAM.Config}}' kpiocqticjlx
```
If the range is the same as your node IPs, you need to change it.

```bash
docker network rm ingress
```

now create a new ingress defining a different ip range

```bash
docker network create --driver overlay --ingress --subnet 172.16.0.0/16 --gateway 172.16.0.1 ingress
```

Restart all your nodes.
