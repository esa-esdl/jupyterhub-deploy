From jupyterhub/singleuser
MAINTAINER Brockmann Consult GmbH <http://www.brockmann-consult.de>
# Install python modules
RUN conda install --quiet --yes 'xarray' 'dask'
RUN conda install -c forman gridtools
RUN conda install -c anaconda --yes basemap=1.0.7
RUN conda install -c conda-forge ffmpeg=2.8.6
USER root
# Install cablab
WORKDIR /srv/jupyterhub
RUN git clone https://github.com/CAB-LAB/cablab-core.git
WORKDIR /srv/jupyterhub/cablab-core
RUN python setup.py install

# Install Julia
WORKDIR /srv/jupyterhub
RUN wget https://julialang.s3.amazonaws.com/bin/linux/x64/0.5/julia-0.5.0-linux-x86_64.tar.gz
RUN mkdir julia && tar xzf julia-0.5.0-linux-x86_64.tar.gz -C julia --strip-components 1
RUN PATH="/srv/jupyterhub/julia/bin:$PATH"
WORKDIR /srv/jupyterhub/julia
RUN echo 'Pkg.add("IJulia")' > IJulia.jl
RUN echo 'Pkg.clone("https://github.com/CAB-LAB/CABLAB.jl");Pkg.checkout("Patchwork")' > installcablab.jl
RUN echo 'Pkg.clone("https://github.com/milanflach/LIBSVM.jl");Pkg.checkout("LIBSVM","mutating_versions")' > installlibsvm.jl
RUN echo 'Pkg.clone("https://github.com/milanflach/MultivariateAnomalies.jl")' > installmultivariate.jl
RUN echo 'using CABLAB' > precomp.jl
USER jovyan
RUN export CABLAB_WORKDIR="/home/jovyan/work/tmp"
RUN /srv/jupyterhub/julia/bin/julia IJulia.jl
RUN /srv/jupyterhub/julia/bin/julia installlibsvm.jl
RUN /srv/jupyterhub/julia/bin/julia installmultivariate.jl
RUN /srv/jupyterhub/julia/bin/julia installcablab.jl
RUN /srv/jupyterhub/julia/bin/julia precomp.jl

USER jovyan
WORKDIR /home/jovyan/work
