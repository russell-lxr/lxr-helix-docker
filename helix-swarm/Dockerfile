# --------------------------------------------------------------------------------
# Docker configuration for Swarm
# --------------------------------------------------------------------------------

#FROM ubuntu:xenial

#LABEL vendor="Perforce Software"
#LABEL maintainer="Paul Allen (pallen@perforce.com)"

# Update Ubuntu and add Perforce Package Source
#RUN \
#  apt-get update  && \
#  apt-get install -y wget curl unzip vim && \
#  wget -qO - https://package.perforce.com/perforce.pubkey | apt-key add - && \
#  echo "deb http://package.perforce.com/apt/ubuntu xenial release" > /etc/apt/sources.list.d/perforce.list && \
#  apt-get update


FROM ubuntu:bionic

# Update package lists
RUN apt-get update

# Install essential tools and gnupg
RUN apt-get install -y wget curl unzip vim gnupg

# Add Perforce public key
RUN wget -qO - https://package.perforce.com/perforce.pubkey | apt-key add -

# Add Perforce repository for Bionic
RUN echo "deb http://package.perforce.com/apt/ubuntu bionic release" > /etc/apt/sources.list.d/perforce.list

# Update package lists to include Perforce packages
RUN apt-get update

# Set environment variables to configure tzdata non-interactively
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/London

# Install tzdata to avoid timezone prompt
RUN apt-get install -y --no-install-recommends tzdata

# Configure timezone settings
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install Helix CLI
RUN apt-get install -y --no-install-recommends helix-cli

# Clean up package lists to reduce image size
RUN rm -rf /var/lib/apt/lists/*

# Create swarm user
RUN useradd --user-group --home-dir /home/swarm --create-home --uid 1000 swarm

# Update package lists again (to ensure latest for next install)
RUN apt-get update

# Install Helix Swarm
RUN apt-get install -y --no-install-recommends helix-swarm

# Final cleanup of package lists
RUN rm -rf /var/lib/apt/lists/*


# --------------------------------------------------------------------------------
# Docker ENVIRONMENT
# --------------------------------------------------------------------------------

# Default Environment
ARG P4PORT=p4d.helix:1666
ARG P4USER=super
ARG P4PASSWD=Passw0rd
ARG SWARMHOST=swarm.helix
ARG SWARMUSER=swarm
ARG SWARMPASSWD=$P4PASSWD
ARG SWARMTOKEN=00000000-0000-0000-0000-000000000000
ARG MAILHOST=swarm.helix

# Configure Swarm Environment
ENV P4PORT=$P4PORT \
  P4USER=$P4USER \
  P4PASSWD=$P4PASSWD \
  SWARMHOST=$SWARMHOST \
  SWARMUSER=$SWARMUSER \
  SWARMPASSWD=$SWARMPASSWD \
  SWARMTOKEN=$SWARMTOKEN \
  MAILHOST=$MAILHOST

EXPOSE 80

# Add external files
COPY files/setup.sh /opt/perforce/swarm/setup.sh
COPY files/config.php /opt/perforce/swarm/data/config.php

# Configure Apache/Swarm
RUN \
	chmod +x /opt/perforce/swarm/setup.sh && \
	chown -R www-data:www-data /opt/perforce/swarm/data && \
	a2dissite 000-default && \
	mkdir -p /opt/perforce/swarm/data/queue/tokens && \
	touch /opt/perforce/swarm/data/queue/tokens/$SWARMTOKEN

USER swarm
COPY files/swarm.p4s /home/swarm/swarm.p4s
COPY files/client.p4s /home/swarm/client.p4s
COPY files/swarm-trigger.pl /home/swarm/swarm-trigger.pl
COPY files/swarm-trigger.conf /home/swarm/swarm-trigger.conf

# --------------------------------------------------------------------------------
# Docker RUN
# --------------------------------------------------------------------------------

USER root
ENTRYPOINT \
    until p4 info -s 2> /dev/null; do sleep 1; done && \
	echo "Perforce Server [FOUND]" && \
	until p4 -p build.helix:1666 info -s 2> /dev/null; do sleep 1; done && \
	echo "Build Server [FOUND]" && \
	cron start && \
	/opt/perforce/swarm/setup.sh && \
	bash

HEALTHCHECK \
	--interval=2m \
	--timeout=10s \
	CMD curl --fail http://localhost/api/version || exit 1
