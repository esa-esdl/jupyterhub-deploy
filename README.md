# jupyterhub-deploy

This is another variant of jupyterhub-deploy, which originally comes from https://github.com/jupyterhub/jupyterhub-deploy-docker. As in the original version, it uses ``make build`` command to create the docker image and ``docker-compose up`` to run the docker container. This version has been successfully deployed and run in 3 Ubuntu VMs (1 for the server and 2 for the nodes).

## Use case scenario
A Jupyterhub server that can spawn individual Jupyter Notebook containers in a cluster. This is to provide a framework for users of Cab-Lab to play around with the data cube. 

## Pre-requisites
1. 3 Ubuntu VMs with docker installed
2. Create GitHub application [here](https://github.com/settings/applications/new) to get the Client ID, Client Secret, and Authorization callback URL. Modify the .env file with these information.
3. TLS certificates ([self-signed](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#using-ssl-for-encrypted-communication) or [letsencrypt](https://letsencrypt.org/))

## Pre-configuration
1. `cd jupyterhub-deploy`
2. `cp .env.example .env`
3. Modify .env

## Deployment
1. **VM1** acts as a Jupyterhub server. Therefore, 2 components need to be set up: docker swarm manager and docker swarm consul. More information about those two can be found [here](https://docs.docker.com/swarm/install-manual/).
  * run the consul container  
     ```docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap```
  * run the swarm manager  
      ```docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise [VM1 host]:4000 consul://[VM1 host]:8500```
2. **VM2** acts as a node1 as well as a replica manager. 
  * run the swarm manager  
     ```docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise [VM2 host]:4000 consul://[VM1 host]:8500```
  * run the swarm node  
     ```docker run -d --net swarmnet swarm join --advertise=[VM2 host]:2375 consul://[VM1 host]:8500```
3. **VM3** acts as a node2.
  * run the swarm node  
     ```docker run -d --net swarmnet swarm join --advertise=[VM3 host]:2375 consul://[VM1 host]:8500```
