#FROM ubuntu:xenial

#LABEL vendor="Perforce Software"
#LABEL maintainer="Paul Allen (pallen@perforce.com)"

# Update Ubuntu and add Perforce Package Source
#RUN \
#  apt-get update && \
#  apt-get install -y wget unzip vim netcat lsb-core net-tools rsyslog && \
#  apt-get install -y openjdk-8-jdk && \
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

RUN apt-get install -y helix-cli

ADD http://ftp.perforce.com/perforce/r22.3/bin.java/helix-p4search.tgz /p4search.tgz  

RUN tar xfz /p4search.tgz

## TODO - add triggers:
#  SEARCH.change    form-commit   change "%//depot/p4search/install/search-queue.sh% -t change   -v %formname%"
#  SEARCH.commit    change-commit //...  "%//depot/p4search/install/search-queue.sh% -t commit   -v %change%"
