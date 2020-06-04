# IOTA Node

[![Hornet](https://img.shields.io/badge/hornet-0.4.0--rc11-blue)](https://github.com/gohornet/hornet/releases/tag/v0.4.0-rc11)

Configuration files for launching a iota node on a cloud server. 

This is for
- A single node
- Multiple nodes connected to each other
- A private node network

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

The following software needs to be installed

- docker
- docker-compose
- git

### Installing

Before installing, ensure the following ports are open. This is both in the firewall software running on the server and on the firewall provided by the cloud service provider, if the node is being run in the cloud.

```
14626 UDP - Autopeering port
15600 TCP - Gossip (neighbors) port
14265 TCP - API port (optional if you don't want to access your node's API from external)
```

Instructions on how to set up docker, docker-commpose and ufw (on Ubuntu Server 18.04) can be found here
- Docker https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
- Docker-compose https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04
- UFW (Uncomplicated Firewall) https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04

#### Single node

The first step is to clone the repository by running:

```
git clone https://github.com/peterokwara/iota-node.git
```

With the repository cloned, navigate to the `iota-node/hornet/` directory and run:

```
docker-compose up -d
```

This allows one to run docker in a detached mode. To view the logs, run:

```
docker-compose logs -f
```

This streams the log file of what is happening within the docker file. 

To shutdown the node, you can simply run:

```
docker-compose down
```

#### Multiple nodes connected to each other

The first step is to clone the repository by running:

```
git clone https://github.com/peterokwara/iota-node.git
```

With the repository cloned, navigate to the `iota-node/hornet/` directory.

Modify the `docker-compose.yml` file by adding the following line on the command section.

```
command: ["hornet", "-c", "/home/ubuntu/hornet/config/config", "-n", "/home/ubuntu/hornet/config/peering"]
```

So that the `docker-compose.yml` becomes:

```
version: '3.4'
services:
  hornet:
    build:
      context: .
      dockerfile: Dockerfile
    image: iota/hornet
    command: ["hornet", "-c", "/home/ubuntu/hornet/config/config", "-n", "/home/ubuntu/hornet/config/peering"]
    volumes:
      - ./config:/home/ubuntu/hornet/config
    ports:
     - "14265:14265"
     - "15600:15600"
     - "14626:14626"
```

To add neighbours, go to the `iota-node/hornet/config/peering.json` and add the url and the port number of the other nodes you need to connect to under identity. 

```
{
  "acceptAnyConnection": false,
  "maxPeers": 5,
  "peers": [
    {
      "identity": "example.neighbor.com:15600",
      "alias": "Example Peer",
      "preferIPv6": false
    }
  ]
}
```

Multiple nodes can be added by just adding another json object shown below

```
{
  "acceptAnyConnection": false,
  "maxPeers": 5,
  "peers": [
    {
      "identity": "example.neighbor.com:15600",
      "alias": "Example Peer",
      "preferIPv6": false
    },
    {
      "identity": "example.neighbor.com:15600",
      "alias": "Example Peer",
      "preferIPv6": false
    }
  ]
}
```

Make sure the node address and port numbers are added on both sides for the nodes to successfully talk to each other.

Once this is done, you can now move into the `iota-node/hornet` folder and run:

```
docker-compose up -d
```

This allows one to run docker in a detached mode. To view the logs, run:

```
docker-compose logs -f
```

This streams the log file of what is happening within the docker file. 

To shutdown the node, you can simply run:

```
docker-compose down
```
#### Dashboard

**Warning** This is not advisable since anyone can see all the information about your node running. Setting up a username and password to access the dashboard is advisable.

To view the dashboard from anywhere, you need to enable port 8081 on your docker instance. This is done by adding the following line to your docker-compose file in the `iota-node/hornet` path

```
     - "8081:8081"
```

So that your `docker-compose.yml` file becomes:

```
version: '3.4'
services:
  hornet:
    build:
      context: .
      dockerfile: Dockerfile
    image: iota/hornet
    command: ["hornet", "-c", "/home/ubuntu/hornet/config/config", "-n", "/home/ubuntu/hornet/config/peering"]
    volumes:
      - ./config:/home/ubuntu/hornet/config
    ports:
     - "14265:14265"
     - "15600:15600"
     - "14626:14626"
     - "8081:8081"
```
The next step is to modify the `config.json` file in the `iota-node/hornet/config` path by changing the dashboard configuration from

```
"dashboard": {
    "bindAddress": "localhost:8081",
    "theme": "default",
    "dev": false,
    "basicAuth": {
      "enabled": false,
      "username": "",
      "passwordHash": "",
      "passwordSalt": ""
    }
  }
```

to 

```
"dashboard": {
    "bindAddress": "0.0.0.0:8081",
    "theme": "default",
    "dev": false,
    "basicAuth": {
      "enabled": false,
      "username": "",
      "passwordHash": "",
      "passwordSalt": ""
    }
  }
```

by changing the bindAddress to `0.0.0.0` allows the dashboard to be accessed by anywhere in the world.

The next step is to ensure that the port

```
8081 TCP - Dashboard
```

Is open both in the internal firewall in the server, and both in the cloud service provider.

## Built With

* [GoHornet](https://github.com/gohornet/hornet) - The IOTA fullnode software 
* [Compass](https://github.com/iotaledger/compass) - The IOTA Network coordinator

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

