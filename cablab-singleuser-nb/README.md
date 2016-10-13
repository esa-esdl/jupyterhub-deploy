## cablab/singleuser Dockerfile


This repository contains **Dockerfile** of a single user Jupyter notebook that is pre-configured specifically for Cab-Lab usage.


### Base Docker Image

* [jupyterhub/singleuser](https://hub.docker.com/r/jupyterhub/singleuser/)

### What's available on the Jupyter Notebook?

* Python2 and Python3 bundled in Conda installation (more detailed on the included packages can be found in [scipy-notebook Dockerfile](https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile)) 

* cablab-core (latest)

* gridtools (latest)

* Julia 0.4.6

* [future] R


### Installation

1. Install [Docker](https://www.docker.com/).
 
2. `docker build -t cablab/singleuser .`
