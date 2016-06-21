From jupyterhub/singleuser

MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>

# Install python modules
RUN conda install --quiet --yes 'xarray' 'dask'
RUN conda install -c anaconda basemap=1.0.7

USER root

# Install gridtools
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/gridtools.git
WORKDIR /srv/jupyterhub/gridtools
RUN python setup.py install

# Install cablab
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/cablab-core.git
WORKDIR /srv/jupyterhub/cablab-core
RUN python setup.py install

USER jovyan

# Install Julia
#COPY julia-0.4.6 /srv/jupyterhub/julia
WORKDIR /srv/jupyterhub
RUN wget https://julialang.s3.amazonaws.com/bin/linux/x64/0.4/julia-0.4.6-linux-x86_64.tar.gz
RUN mkdir julia && tar xzf julia-0.4.6-linux-x86_64.tar.gz -C julia --strip-components 1
RUN PATH="/srv/jupyterhub/julia/bin:$PATH"
COPY IJulia.jl /srv/jupyterhub/julia
WORKDIR /srv/jupyterhub/julia
RUN /srv/jupyterhub/julia/bin/julia IJulia.jl
