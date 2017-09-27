FROM ubuntu:16.04

######################################################################################################################################################
# Set up
######################################################################################################################################################
RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y software-properties-common vim wget curl unzip zip build-essential python git bash-completion \
 && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

RUN apt-get clean && apt-get update && apt-get install -y locales

######################################################################################################################################################
# Set up Java(lastest)
######################################################################################################################################################
ENV JAVA_HOME       /usr/lib/jvm/java-8-oracle
ENV DEBIAN_FRONTEND noninteractive


## UTF-8
RUN locale-gen en_US.UTF-8
ENV LANG       en_US.UTF-8
ENV LC_ALL     en_US.UTF-8

RUN apt-get update && \
  apt-get dist-upgrade -y

## Remove any existing JDKs
RUN apt-get --purge remove openjdk*

## Install Oracle's JDK
RUN echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | debconf-set-selections
RUN echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" > /etc/apt/sources.list.d/webupd8team-java-trusty.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
RUN apt-get update && \
  apt-get install -y --no-install-recommends oracle-java8-installer && \
  apt-get clean all


######################################################################################################################################################
# systemd
######################################################################################################################################################

ENV container docker

# Don't start any optional services except for the few we need.
RUN find /etc/systemd/system \
         /lib/systemd/system \
         -path '*.wants/*' \
         -not -name '*journald*' \
         -not -name '*systemd-tmpfiles*' \
         -not -name '*systemd-user-sessions*' \
         -exec rm \{} \;

RUN systemctl set-default multi-user.target


######################################################################################################################################################
# Set up Tomcat
######################################################################################################################################################
RUN apt-get update && \
    apt-get install -yq --no-install-recommends wget pwgen ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV TOMCAT_MAJOR_VERSION 8
ENV TOMCAT_MINOR_VERSION 8.5.5
ENV CATALINA_HOME /tomcat

# INSTALL TOMCAT
RUN wget -q https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz && \
    wget -qO- https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz.md5 | md5sum -c - && \
    tar zxf apache-tomcat-*.tar.gz && \
    rm apache-tomcat-*.tar.gz && \
    mv apache-tomcat* /tomcat

RUN chmod -R 777 /tomcat/webapps

RUN \
  apt-get update && \
  apt-get autoremove -y && \
  apt-get clean allEXPOSE 8080

RUN groupadd tomcat
RUN useradd -s /bin/false -g tomcat -d /tomcat tomcat

RUN chgrp -R tomcat /tomcat
RUN chmod -R g+x /tomcat/conf

RUN chmod -R g+r /tomcat
RUN chmod -R g+w /tomcat

COPY tomcat.service /etc/systemd/system/

EXPOSE 8080

######################################################################################################################################################
# Set up ssh
######################################################################################################################################################

RUN apt-get install -y openssh-server net-tools

RUN mkdir /var/run/sshd

RUN echo 'root:root' |chpasswd

RUN echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

EXPOSE 22

CMD ["/sbin/init"]
