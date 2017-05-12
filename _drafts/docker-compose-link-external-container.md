---
layout: post
title: docker-compose how to link external container
excerpt: ""
---

### Create network
~~~~bash
docker network create multi-host-network 
~~~~

### Start up external container in exist network
~~~~bash
docker run -itd --network=multi-host-network db
~~~~

### Add external exist network in docker-compose
~~~~yml
networks:
  default:
    external:
      name: multi-host-network
~~~~

### Link container outside docker-compose
~~~~yml
links:
	- db
~~~~
