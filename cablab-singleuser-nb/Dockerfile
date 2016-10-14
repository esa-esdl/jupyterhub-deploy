From jupyterhub/singleuser

MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>

# Install python modules
RUN conda install --quiet --yes 'xarray' 'dask'
RUN conda install -c forman gridtools
RUN conda install -c anaconda basemap=1.0.7

USER root

# Install cablab
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/cablab-core.git
WORKDIR /srv/jupyterhub/cablab-core
RUN python setup.py install

# Install Julia
RUN apt-get update
RUN apt-get -y install libhdf5-dev
RUN apt-get -y install libnetcdf-dev
WORKDIR /srv/jupyterhub
RUN wget https://julialang.s3.amazonaws.com/bin/linux/x64/0.5/julia-0.5.0-linux-x86_64.tar.gz
RUN mkdir julia && tar xzf julia-0.5.0-linux-x86_64.tar.gz -C julia --strip-components 1
RUN PATH="/srv/jupyterhub/julia/bin:$PATH"
WORKDIR /srv/jupyterhub/julia
RUN echo 'Pkg.add("IJulia")' > IJulia.jl
RUN echo 'Pkg.clone("https://github.com/CAB-LAB/CABLAB.jl")' > installcablab.jl
RUN echo 'Pkg.clone("https://github.com/milanflach/LIBSVM.jl");Pkg.checkout("LIBSVM","mutating_versions")' > installlibsvm.jl
RUN echo 'Pkg.clone("https://github.com/milanflach/MultivariateAnomalies.jl")' > installmultivariate.jl
RUN echo 'using CABLAB' > precomp.jl
USER jovyan
RUN /srv/jupyterhub/julia/bin/julia IJulia.jl
RUN /srv/jupyterhub/julia/bin/julia installcablab.jl
RUN /srv/jupyterhub/julia/bin/julia installlibsvm.jl
RUN /srv/jupyterhub/julia/bin/julia installmultivariate.jl
RUN /srv/jupyterhub/julia/bin/julia precomp.jl



WORKDIR /home/jovyan/work
