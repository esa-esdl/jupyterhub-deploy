From jupyterhub/singleuser

MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>

RUN conda install --quiet --yes 'xarray' 'dask'

USER root

COPY gridtools /srv/jupyterhub/gridtools
WORKDIR /srv/jupyterhub/gridtools
RUN python setup.py install

COPY cablab-core /srv/jupyterhub/cablab-core
WORKDIR /srv/jupyterhub/cablab-core
RUN python setup.py install

# Install Julia. Still looking for the best way to install Julia in this container.
#RUN apt-get update
#RUN apt-get -y install software-properties-common
#RUN add-apt-repository -y ppa:staticfloat/juliareleases
#RUN add-apt-repository -y ppa:staticfloat/julia-deps
#RUN apt-get update
#RUN apt-get install -y julia
#RUN rm -rf /var/lib/apt/lists/*
