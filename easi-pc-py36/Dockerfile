FROM ubuntu:18.04

LABEL maintainer="CSIRO EASI Data Cube Training on PC <Robert.Woodcock@csiro.au>"

USER root

# A littel apt configuration to speed downloads a little
ADD 99parallel /etc/apt/apt.conf.d/99parallel

# Switch to Australian mirror for ubuntu apt packages
RUN sed --in-place --regexp-extended "s/(\/\/)(archive\.ubuntu)/\1au.\2/" /etc/apt/sources.list && \
	apt-get update && apt-get upgrade --yes

ENV DEBIAN_FRONTEND noninteractive

# First add the NextGIS repo
RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

RUN add-apt-repository ppa:nextgis/ppa

RUN apt-get update && \
	apt-get install -y --no-install-recommends \
    # Core requirements from travis.yml
    gdal-bin gdal-data libgdal-dev libgdal20 libudunits2-0 \
    # Extra python components, to speed things up
    python3 python3-setuptools python3-dev \
    python3-numpy python3-netcdf4 python3-gdal \
    # Need pip to install more python packages later.
    # The libdpkg-perl is needed to build pyproj
    python3-pip python3-wheel libdpkg-perl \
    # Git to work out the ODC version number
    git \
    # G++ because GDAL decided it needed compiling
    g++ \
    # numpy requires headers for cf_units
    libudunits2-dev \
    # Additional utilities useful for ODC dev
	postgresql-client \
	curl \
    wget \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    ffmpeg \
    # OS dependencies for fully functional Jupyter notebook server
    build-essential \
    emacs \
    inkscape \
    jed \
    libsm6 \
    libxext-dev \
    libxrender1 \
    lmodern \
    netcat \
    pandoc \
    texlive-fonts-extra \
    texlive-fonts-recommended \
    texlive-generic-recommended \
    texlive-latex-base \
    texlive-latex-extra \
    texlive-xetex \
    unzip \
    nano \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Configure Jupyter Notebook
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER=jovyan \
    NB_UID=1000 \
    NB_GID=100 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV HOME=/home/$NB_USER

ADD fix-permissions /usr/local/bin/fix-permissions
# Create jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN groupadd wheel -g 11 && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME

# Install Tini.. this is required because CMD (below) doesn't play nice with notebooks for some reason: https://github.com/ipython/ipython/issues/7062, https://github.com/jupyter/notebook/issues/334
RUN curl -L https://github.com/krallin/tini/releases/download/v0.10.0/tini > tini && \
    echo "1361527f39190a7338a0b434bd8c88ff7233ce7b9a4876f3315c22fce7eca1b0 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

USER $NB_UID

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work && \
    mkdir /home/${NB_USER}/odc && \
    fix-permissions /home/$NB_USER

# Install psycopg2 as a special case, to quiet the warning message 
RUN pip3 install --no-cache --no-binary :all: psycopg2

# Install jupyter notebook and additional python packages for interactive odc use
RUN pip3 install jupyter matplotlib folium ffmpeg shapely scikit-image

RUN ~/.local/bin/jupyter notebook --generate-config && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions /home/$NB_USER

USER root

EXPOSE 8888

# Configure container startup
ENTRYPOINT ["/usr/local/bin/tini", "-g", "--"]

# launch notebook
CMD ["start-notebook.sh"]

# Add local files as late as possible to avoid cache busting
COPY start.sh /usr/local/bin/
COPY start-notebook.sh /usr/local/bin/
COPY start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/
RUN fix-permissions /etc/jupyter/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
