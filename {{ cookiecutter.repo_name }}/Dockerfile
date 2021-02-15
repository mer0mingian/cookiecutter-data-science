# Data Science Project to production base image

# Ubuntu 20.04 (focal)
# https://hub.docker.com/_/ubuntu/?tab=tags&name=focal
# OS/ARCH: linux/amd64

ARG ROOT_CONTAINER=ubuntu:focal
ARG BASE_CONTAINER=$ROOT_CONTAINER
FROM $BASE_CONTAINER

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# -----------------------------------------------------------------------------------------------------------
# Base setup and apt installations

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -yq --no-install-recommends git openssh-client xz-utils neovim make


# -----------------------------------------------------------------------------------------------------------
# Anaconda

RUN mkdir -p /opt/conda
COPY --from=continuumio/miniconda3:4.8.3 /opt/conda /opt/conda
RUN ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc
WORKDIR $HOME

# Make conda findable, configure and install packages
RUN export "PATH=/opt/conda/bin:$PATH" && \
    conda init bash && \
    # get important channels
    conda config --system --add channels conda-forge && \
    conda config --system --add channels pytorch && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda config --system --set channel_priority strict && \
    # Get Python, Conda, Pip & Tini, pin them and update environment
    conda install --quiet --yes \
        python=3.8 conda pip 'tini=0.18.0' numpy pandas>=1.0.2
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> /opt/conda/conda-meta/pinned && \
    conda list tini | grep tini | tr -s ' ' | cut -d ' ' -f 1,2 >> /opt/conda/conda-meta/pinned && \
    conda update --all --quiet --yes && \
    # Clean up
    conda clean --all -f -y

# -----------------------------------------------------------------------------------------------------------
# Start SSH Agent on boot

WORKDIR /home/root
COPY ssh_agent_setup.txt .
RUN cat ./ssh_agent_setup.txt >> .bashrc


# Enable prompt color in the skeleton .bashrc
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Keep the image small
RUN apt-get clean \
    && apt-get auto-remove -y \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

# -----------------------------------------------------------------------------------------------------------
# Start SSH Agent on boot
WORKDIR /home/root
COPY . .
# WORKDIR /home/root
# RUN pip install --force-reinstall -e .

# -----------------------------------------------------------------------------------------------------------
# Define entrypoint
# SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# CMD [ "/bin/bash" ]
