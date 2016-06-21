## cablab/singleuser Dockerfile


This repository contains **Dockerfile** of a single user Jupyter notebook that is pre-configured specifically for Cab-Lab usage.


### Base Docker Image

* [jupyterhub/singleuser](https://hub.docker.com/r/jupyterhub/singleuser/)

### What's available on the Jupyter Notebook?

* Python2 and Python3 bundled in Conda installation (more detailed on the included packages can be found in [scipy-notebook Dockerfile](https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile)) 

* cablab module with gridtools (after completing the following installation part)

* Julia

* [future] R


### Installation

1. Install [Docker](https://www.docker.com/).
 
2. `cd cablab-singleuser-nb`

3. `git clone https://github.com/CAB-LAB/gridtools.git`

4. `git clone https://github.com/CAB-LAB/cablab-core.git`
 
5. `wget https://julialang.s3.amazonaws.com/bin/linux/x64/0.4/julia-0.4.6-linux-x86_64.tar.gz`
 
6. `mkdir julia-0.4.6 && sudo tar xzf julia-0.4.6-linux-x86_64.tar.gz -C julia-0.4.6 --strip-components 1`

7. `docker build -t cablab/singleuser .`
