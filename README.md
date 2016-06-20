# jupyterhub-deploy

This is another variant of jupyterhub-deploy, which originally comes from https://github.com/jupyterhub/jupyterhub-deploy-docker. As in the original version, it uses ``make build`` command to create the docker image and ``docker-compose up`` to run the docker container. This version has been successfully deployed and run in 3 Ubuntu VMs (1 for the server and 2 for the nodes).

## Use case scenario
A Jupyterhub server that can spawn individual Jupyter Notebook containers in a cluster. This is to provide a framework for users of Cab-Lab to play around with the data cube. 

## Pre-requisites
1. 3 Ubuntu VMs with docker installed
2. Create GitHub application [here](https://github.com/settings/applications/new) to get the Client ID, Client Secret, and Authorization callback URL. Modify the .env file with these information.
3. TLS certificates ([self-signed](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#using-ssl-for-encrypted-communication) or [letsencrypt](https://letsencrypt.org/))

## Pre-configuration
1. Modify `.env`file  
  * `cd jupyterhub-deploy`  
  * `cp .env.example .env`  
  * modify .env
2. Modify `userlist` file to grant normal or admin access to GitHub account(s)  
  * `cd jupyterhub-deploy`  
  * `touch userlist`
  * modify userlist  
     Example :  
     user1 admin  
     user2  
     user3

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
     ```docker run -d swarm join --advertise=[VM2 host]:2375 consul://[VM1 host]:8500```
3. **VM3** acts as a node2.
  * run the swarm node  
     ```docker run -d swarm join --advertise=[VM3 host]:2375 consul://[VM1 host]:8500```
4. Restart the docker daemon in each VM with additional arguments to allow it to be part of the swarm cluster  
  * VM1 : `nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM1 host]:2375 --cluster-store consul://[VM1 host]:8500 &`  
  * VM2 : `nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM2 host]:2375 --cluster-store consul://[VM1 host]:8500 &`  
  * VM3 : `nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM3 host]:2375 --cluster-store consul://[VM1 host]:8500 &`
5. Create an overlay docker network so that containers can communicate to each other.  
   In VM1 : ```docker -H tcp://0.0.0.0:4000 network create -d overlay swarmnet```  
   Now **swarmnet** network should be available in all 3 VMs. Check using this command `docker network ls`
6. Setup a NFS server in VM1 (for more information go to [this page](http://www.tldp.org/HOWTO/NFS-HOWTO/server.html)  
  * ``vim /etc/exports`` and add the following entries  
 ```
 /_[any path]_/jupyterhub-shared    *(rw,sync,no_root_squash)  
 /_[any path]_/cablab-shared        *(rw,sync,no_root_squash)
 ```
  * ``exportfs -r``
7. Mount **jupyterhub-shared** and **cablab-shared** in each node VM  
  * `mount [VM1 host]:/_[any path]_/jupyterhub-shared /var/lib/docker/volumes`  
  * `mount [VM1 host]:/_[any path]_/cablab-shared /_[any local path]_/cablab-shared`
8. Install cablab package  
  * `cd /_[any local path]_/cablab-shared`  
  * `git clone https://github.com/CAB-LAB/gridtools.git`  
  * `git clone https://github.com/CAB-LAB/cablab-core.git`
9. Create cablab/singleuser docker image  
  * `cd /_[any local path]_/cablab-shared`  
  * `docker build -t cablab/singleuser .`  
10. Make sure that `.env` file contains the correct information (in case of any name customisations). 
11. `cd jupyterhub-deploy`
12. `make build`
13. `docker-compose up -d`
14. Open a browser and go to `https://[VM1 host]`
