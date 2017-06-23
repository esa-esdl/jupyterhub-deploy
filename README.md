# jupyterhub-deploy

This is another variant of jupyterhub-deploy, which originally comes from https://github.com/jupyterhub/jupyterhub-deploy-docker. The overall architecture has been inspired by the original jupyterhub-deploy as well as a very [informative article](https://developer.rackspace.com/blog/deploying-jupyterhub-for-education/) by Jessica Hamrick ([@jhamrick](https://github.com/jhamrick)). As in the original version, it uses ``make build`` command to create the docker image and ``docker-compose up`` to run the docker container. This version has been successfully deployed and run in 3 Ubuntu VMs (1 for the server and 2 for the nodes).

# Table of Contents
  * [Use case scenario](#use-case-scenario)
  * [Pre-requisites](#pre-requisites)
  * [Pre-configuration](#pre-configuration)
  * [Deployment Ubuntu](#deployment-ubuntu)
  * [Deployment CentOS](#deployment-centos)
    * [Pre-configuration](#pre-configuration-for-devicemapper-storage-driver)
    * [Deployment on VM1](#deployment-on-vm1)
    * [Deployment on VM2](#deployment-on-vm2)
    * [Deployment on VM3](#deployment-on-vm3)

## Use case scenario
A Jupyterhub server that can spawn individual Jupyter Notebook containers in a cluster. This is to provide a framework for users of Cab-Lab to play around with the data cube. 

## Pre-requisites
1. 3 Ubuntu VMs with docker and docker-compose installed
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
     <pre><code>user1 admin  
     user2  
     user3</pre></code>
3. Copy the certificates to `secrets`folder
  * `cd jupyterhub-deploy` 
  * mkdir secrets
  * `cp <any directory>/jupyterhub.crt secrets/`
  * `cp <any directory>/jupyterhub.key secrets/`
4. Create a cookie secret, which is an encryption key, used to encrypt the browser cookies used for authentication.
  * `cd jupyterhub-deploy` 
  * `openssl rand -hex 32 > cookie_secret`

## Deployment Ubuntu
1. **VM1** acts as a Jupyterhub server. Therefore, 2 components need to be set up: docker swarm manager and docker swarm consul. More information about those two can be found [here](https://docs.docker.com/swarm/install-manual/).
  * run the consul container  
     ```docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap```
  * run the swarm manager  
      ```docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise [VM1 host]:4000 consul://[VM1 host]:8500```
2. **VM2** acts as a node1 as well as a second manager. 
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
 <pre><code>/_[any path]_/jupyterhub-shared    *(rw,sync,no_root_squash)  
 /_[any path]_/cablab-shared        *(rw,sync,no_root_squash)</pre></code>
  * ``exportfs -r``
7. Mount **jupyterhub-shared** and **cablab-shared** in each node VM  
   <pre><code>mount _[VM1 host]_:/_[any path]_/jupyterhub-shared /var/lib/docker/volumes
   mount _[VM1 host]_:/_[any path]_/cablab-shared /_[any local path]_/cablab-shared</code></pre>
8. Create cablab/singleuser docker image  
  <pre><code>cd /_[any local path]_/cablab-shared
  docker build -t cablab/singleuser .</pre></code>
9. Make sure that `.env` file contains the correct information (in case of any name customisations). 
10. `cd jupyterhub-deploy`
11. `make build`
12. `docker-compose up -d`
13. Open a browser and go to `https://[VM1 host]`

## Deployment CentOS
This instruction will assume 3 VMs are available for the set-up. **VM1** will act as the Jupyterhub server, the docker swarm main manager, as well as the consul. **VM2** will act as the first node and **VM3** will act as the second node.
### Pre-configuration (for devicemapper storage driver)
This pre-configuration steps are not necessarily for only CentOS deployment, but for all deployment in an environment where the storage driver is devicemapper. In this environment, when running docker daemon, by default it uses loop-lvm mode. This mode uses sparse files to build the thin pool used by image and container snapshots and it is not very efficient for extensive IO operations within the containers. [Docker](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#/configure-docker-with-devicemapper)  states that this mode is not suitable for production use and recommends direct-lvm mode instead. So here are the steps on how to configure a direct-lvm mode in CentOS VM. These have been tested in CentOS 7.2 with kernel 3.10.

1. Create docker Volume Group
<pre><code>sudo yum install -y lvm2*
sudo pvcreate /dev/vdb
sudo vgcreate docker /dev/vdb
sudo lvcreate --wipesignatures y -n thinpool docker -l 95%VG
sudo lvcreate --wipesignatures y -n thinpoolmeta docker -l 1%VG
sudo lvconvert -y --zero n -c 512K --thinpool docker/thinpool --poolmetadata docker/thinpoolmeta
</pre></code>
2. To change the thinpool profile, modify /etc/lvm/profile/docker-thinpool.profile to
<pre><code>activation {
    thin_pool_autoextend_threshold=80
    thin_pool_autoextend_percent=20
}
</pre></code>
3. Activate the new profile
<pre><code>sudo lvchange --metadataprofile docker-thinpool docker/thinpool
</pre></code>
4. Install Docker version 1.11
<pre><code>sudo yum update
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
sudo yum install docker-engine-1.11.2
</pre></code>
6. Start docker daemon
<pre><code>nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM1 host]:2375 --cluster-store consul://[VM1 host]:8500 -s devicemapper --storage-opt dm.thinpooldev=/dev/mapper/docker-thinpool --storage-opt dm.use_deferred_removal=true &
</pre></code>

### Deployment on VM1

1. Install Docker Compose
<pre><code>sudo -i
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
</pre></code>
2. Set-up NFS server
Assumptions: **/data** is the datacube directory, **/container-data** is the docker container directory, **~/cablab-shared** is the directory for sample notebooks
<pre><code>cd ~
sudo mkdir cablab-shared
sudo mkdir /data
sudo mount -o defaults /dev/data/datacube /data (after attaching the datacube volume to this VM)
sudo vim /etc/exports
		[USER_HOME]/cablab-shared        *(rw,sync,no_root_squash)
		/data                            *(rw,sync,no_root_squash)
  /container-data                  *(rw,sync,no_root_squash)
</pre></code>
3. Start docker daemon
<pre><code>nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM1 host]:2375 --cluster-store consul://[VM1 host]:8500 -s devicemapper --storage-opt dm.thinpooldev=/dev/mapper/docker-thinpool --storage-opt dm.use_deferred_removal=true &
</pre></code>
4. Mount the centralised docker volume directory. 
<pre><code>sudo mount [VM1 host]:/container-data /var/lib/docker/volumes
</pre></code>
5. Start consul
<pre><code>docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap
</pre></code>
6. Start docker swarm manager
<pre><code>docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise [VM1 host]:4000 consul://[VM1 host]:8500
</pre></code>

### Deployment on VM2

1. Start docker daemon
<pre><code>nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM2 host]:2375 --cluster-store consul://[VM1 host]:8500 -s devicemapper --storage-opt dm.thinpooldev=/dev/mapper/docker-thinpool --storage-opt dm.use_deferred_removal=true &
</pre></code>
2. Mount centralised directories
<pre><code>sudo mount [VM1 host]:/container-data /var/lib/docker/volumes
sudo mount [VM1 host]:[USER_HOME]/cablab-shared [USER_HOME]/cablab-shared
sudo mount [VM1 host]:/data /data
</pre></code>

### Deployment on VM3

1. Start docker daemon
<pre><code>nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-advertise [VM3 host]:2375 --cluster-store consul://[VM1 host]:8500 -s devicemapper --storage-opt dm.thinpooldev=/dev/mapper/docker-thinpool --storage-opt dm.use_deferred_removal=true &
</pre></code>
2. Mount centralised directories
<pre><code>sudo mount [VM1 host]:/container-data /var/lib/docker/volumes
sudo mount [VM1 host]:[USER_HOME]/cablab-shared [USER_HOME]/cablab-shared
sudo mount [VM1 host]:/data /data
</pre></code>

# SSL Certificate renewal

1. Run `certbot renew` as stated in [here](ttp://letsencrypt.readthedocs.io/en/latest/using.html#renewing-certificates)
2. Replace the existing certs in /secrets with the newly generated ones (the cert*x*.pem and privkey*x*.pem). cert*x*.pem as jupyterhub.crt and privkey*x*.pem as jupyterhub.key
3. `docker rm -f jupyterhub && docker rmi -f jupyterhub && docker-compose up -d`
