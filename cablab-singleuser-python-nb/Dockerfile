From jupyterhub/singleuser
MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>
# Install python modules
RUN conda install --quiet --yes 'xarray' 'dask'
RUN conda install -c cablab gridtools
RUN conda install -c anaconda --yes basemap=1.0.7
RUN conda install -c conda-forge ffmpeg=2.8.6
USER root
# Install esdl
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/esa-esdl/esdl-core.git
WORKDIR /srv/jupyterhub/esdl-core
RUN python setup.py install

USER jovyan
WORKDIR /home/jovyan
