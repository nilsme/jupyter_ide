FROM fedora:36


# Set build arguments #########################################################

# Set nfs server
ARG NFS_SERVER=jupyter_nfs
# Set versions
ARG MINICONDA_VERSION=py38_4.9.2
ARG R_VERSION=4.2.1


# Configure system ############################################################

# Set environment
ENV TZ=Europe/Amsterdam
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

# Install system packages
RUN yum -y update && \
    yum -y install \
    # Install git
    git \
    # Install compilers
    gcc \
    gcc-c++ \
    gcc-gfortran \
    java-latest-openjdk \
    # Install system packages for compilers and R
    readline-devel \
    libX11-devel \
    libICE-devel \
    libXt-devel \
    zlib-devel \
    cairo \
    cairo-devel \
    libcurl \
    libcurl-devel \
    libsodium-devel \
    gsl-devel \
    jq \
    tesseract \
    libpng-devel \
    libtiff-devel \
    unixODBC-devel \
    sqlite-devel \
    postgresql-odbc \
    openssl-devel \
    libxml2 \
    libxml2-devel \
    harfbuzz-devel \
    fribidi-devel \
    libjpeg-turbo-devel \
    zeromq-devel \
    # Install latex
    texlive \
    # Install auxiliary system packages
    wget \
    zip \
    unzip \
    vim \
    nfs-utils


# Install and configure R #####################################################

# Download R and compile R from source
RUN curl https://cran.r-project.org/src/base/R-${R_VERSION:0:1}/R-${R_VERSION}.tar.gz \
    -o /tmp/R-${R_VERSION}.tar.gz && \
    tar -xzvf /tmp/R-${R_VERSION}.tar.gz -C /tmp && \
    cd /tmp/R-${R_VERSION} && \
    ./configure \
    --prefix=/opt/R/${R_VERSION} \
    --enable-memory-profiling \
    --enable-R-shlib \
    --with-blas \
    --with-lapack && \
    make && \
    make install && \
    rm /tmp/R-${R_VERSION}.tar.gz

# Link R
RUN ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R && \
    ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript


# Install Cairo
RUN R -e "install.packages(c('Cairo'), \
                           repo='https://cran.microsoft.com/', \
                           Ncpus=parallel::detectCores())"

# Set device to cairo
RUN echo "options(bitmapType='cairo')" >> \
    /opt/R/${R_VERSION}/lib64/R/etc/Rprofile.site

# Install tidyverse
RUN R -e "install.packages(c('tidyverse'), \
                           repo='https://cran.microsoft.com/', \
                           Ncpus=parallel::detectCores())" 


# Install and configure Python ################################################

# Install miniconda3 for Python support
RUN wget https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh \
    -O /tmp/miniconda.sh  && \
    bash /tmp/miniconda.sh -f -b -p /opt/python/conda && \
    /opt/python/conda/bin/conda update -n base -c conda-forge conda && \
    /opt/python/conda/bin/pip install --upgrade pip && \
    rm /tmp/miniconda.sh

# Init conda for root
RUN /opt/python/conda/bin/conda init && \
    source ~/.bashrc

# Add conda to PATH
ENV PATH=/opt/python/conda/bin:$PATH

# Install Python + NodeJS with conda, Python packages and JupyterHub
RUN /opt/python/conda/bin/conda install --yes -c conda-forge \
      python \
      sqlalchemy \
      tornado \
      jinja2 \
      traitlets \
      requests \
      pycurl \
      nodejs \
      configurable-http-proxy \
      notebook \
      jupyterlab \
      jupyterlab-git \
      jupyterlab-latex \
      jupyter-dash \
      ipywidgets \
      jupyterhub \
      conda-build \
      nb_conda_kernels \
      ipykernel

# Create launcher logo
RUN /opt/python/conda/bin/python -m ipykernel install \
    --user \
    --name python3 \
    --display-name "Python 3"

RUN mkdir -p /usr/local/share/jupyter/kernels/python3 && \
    cp /root/.local/share/jupyter/kernels/python3/* \
    /usr/local/share/jupyter/kernels/python3/

# Remove kernel icon (if any)
RUN rm -rf /opt/python/conda/share/jupyter/kernels/python3

# Build jupyterlab extensions
RUN /opt/python/conda/bin/jupyter lab build

# Create a directory for config
RUN mkdir -p /srv/jupyterhub/

# Copy config
COPY ./jupyterhub_config.py /srv/jupyterhub/jupyterhub_config.py

# Create admin group
RUN groupadd --gid 2000 jupyterhub-admins

# Create basic data science environment
RUN /opt/python/conda/bin/conda create --yes --name py38_ml -c conda-forge \
    python=3.8 \
    pandas \
    matplotlib \
    dash \
    jupyter-dash \
    plotly \
    scikit-learn \
    keras \
    tensorflow \
    ipykernel

# Create launcher logo
RUN /opt/python/conda/envs/py38_ml/bin/python -m ipykernel install \
    --user \
    --name py38_ml \
    --display-name "Python 3.8 (ML)"

RUN mkdir -p /usr/local/share/jupyter/kernels/py38_ml && \
    cp /root/.local/share/jupyter/kernels/py38_ml/* \
    /usr/local/share/jupyter/kernels/py38_ml/

# Remove conda specific kernel logo
RUN rm -rf /opt/python/conda/envs/py38_ml/share/jupyter/kernels/python3/

# Install IRKernel to use R kernel in Jupyter #################################

# Install IRkernel
RUN R -e "install.packages(c('IRkernel', 'repr', 'IRdisplay', 'devtools'), \
                           repo='https://cran.microsoft.com/', \
                           Ncpus=parallel::detectCores())"

RUN R -e "IRkernel::installspec(user=FALSE, name='ir42', displayname='R 4.2')"


# Install IJulia (Julia kernel) ###############################################

# Install Julia
RUN yum -y install julia


# Configure NFS ###############################################################

# Create folder for nfs mount
RUN mkdir -p /mnt/nfs/home/

# Set new default /home for new users
RUN sed -i 's/HOME=\/home/HOME=\/mnt\/nfs\/home/g' /etc/default/useradd

# Mount nfs server on new home directory on reboot
RUN echo "${NFS_SERVER}:/ /mnt/nfs/home/ nfs defaults 0 0" >> /etc/fstab


# Modify .bashrc to run conda init and Julia config at first start ############
COPY ./bashrc /tmp/bashrc
RUN cat /tmp/bashrc >> /etc/skel/.bashrc && \
    rm /tmp/bashrc


# Entrypoint ##################################################################

# Expose standard port for jupyterhub
EXPOSE 8000

ENTRYPOINT mount /mnt/nfs/home && /opt/python/conda/bin/jupyterhub
