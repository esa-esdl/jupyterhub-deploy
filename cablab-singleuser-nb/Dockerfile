From jupyterhub/singleuser

MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>

# Install python modules
RUN conda install --quiet --yes 'xarray' 'dask'

USER root

# Install gridtools
#COPY gridtools /srv/jupyterhub/gridtools
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/gridtools.git
WORKDIR /srv/jupyterhub/gridtools
RUN python setup.py install

# Install cablab
#COPY cablab-core /srv/jupyterhub/cablab-core
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/cablab-core.git
WORKDIR /srv/jupyterhub/cablab-core
RUN python setup.py install

USER jovyan

# Install Julia
COPY julia-0.4.6 /srv/jupyterhub/julia
RUN PATH="/srv/jupyterhub/julia/bin:$PATH"
COPY IJulia.jl /srv/jupyterhub/julia
WORKDIR /srv/jupyterhub/julia
RUN /srv/jupyterhub/julia/bin/julia IJulia.jl
