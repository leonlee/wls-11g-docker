# LICENSE MIT License 2015
#
# GIBAHOLMS DOCKERFILES PROJECT
# -----------------------------------------
# This is the Dockerfile for a default installation for Oracle WebLogic Server 11g (10.3.6) Generic Distribution
# 
# IMPORTANT
# -----------------------------------------
# The resulting image of this Dockerfile DOES NOT contain a WLS Domain.
# For that, you must to create a domain on a new inherited image.
#
# REQUIRED FILES TO BUILD THIS IMAGE
# -----------------------------------------
# (1) wls1036_generic.jar (Oracle WebLogic Server 10.3.6 Generic Installer)
#
# (2) jdk-7u79-linux-x64.rpm (Oracle Java SE Development Kit 7 for Linux x64 RPM)
#
# HOW TO BUILD THIS IMAGE
# -----------------------------------------
# Put all downloaded files in the same directory as this Dockerfile
# Run: 
#      $ sudo docker build -t gibaholms/weblogic:10.3.6 . 
#
# AUTHOR
# -----------------------------------------
# Gilberto Holms <gibaholms85@gmail.com>
# https://gibaholms.wordpress.com/
# -----------------------------------------

# Pull base image
# ---------------
FROM registry.shurenyun.com/centos7/base:omega.v0.2

# Maintainer
# ----------
MAINTAINER Li Guoqiang <gqli@dataman-inc.com>

# Environment variables required for this build (do NOT change)
# -------------------------------------------------------------
ENV JAVA_PKG jrockit-jdk1.6.0_45-R28.2.7-4.1.0-linux-x64.bin 
ENV WLS_PKG wls1036_generic.jar
ENV JAVA_HOME /usr/java/default
ENV PATH $JAVA_HOME/bin:$PATH

# Setup required packages (unzip), filesystem, and oracle user
# ------------------------------------------------------------
RUN umask 027 && \
    mkdir /u01 && \
    chmod a+xr /u01 && \
    useradd -b /u01 -m -s /bin/bash oracle 

# Copy packages
COPY $WLS_PKG /u01/
COPY $JAVA_PKG /u01/
COPY wls-silent.xml /u01/
COPY jrk-silent.xml /u01/

# Install and configure Oracle JDK
# -------------------------------------
RUN /u01/$JAVA_PKG -mode=silent -silent_xml=/u01/jrk-silent.xml -log=/u01/jdk_install.log && \ 
    cat /u01/jdk_install.log && \
    rm /u01/$JAVA_PKG /u01/jrk-silent.xml /u01/jdk_install.log

# Change the open file limits in /etc/security/limits.conf
RUN sed -i '/.*EOF/d' /etc/security/limits.conf && \
    echo "* soft nofile 16384" >> /etc/security/limits.conf && \ 
    echo "* hard nofile 16384" >> /etc/security/limits.conf && \ 
    echo "# EOF"  >> /etc/security/limits.conf

# Change the kernel parameters that need changing.
RUN echo "net.core.rmem_max=4192608" > /u01/oracle/.sysctl.conf && \
    echo "net.core.wmem_max=4192608" >> /u01/oracle/.sysctl.conf && \ 
    sysctl -e -p /u01/oracle/.sysctl.conf

# Adjust file permissions, go to /u01 as user 'oracle' to proceed with WLS installation
RUN chown oracle:oracle -R /u01
WORKDIR /u01
USER oracle

# Installation of WebLogic 
RUN mkdir /u01/tmp && \
    java -jar $WLS_PKG -mode=silent -silent_xml=/u01/wls-silent.xml \
        -log=/u01/server_install.log \
        -Djava.io.tmpdir=/u01/tmp \
        -Djava.security.egd=file:/dev/./urandom && \ 
    cat /u01/server_install.log && \
	rm -rf /u01/$WLS_PKG \
        /u01/wls-silent.xml \
        /u01/server_install.log \
        /u01/tmp && \
    export CONFIG_JVM_ARGS="-Djava.security.egd=file:/dev/./urandom" && \
    export JAVA_OPTIONS=" -Djava.awt.headless=true " && \
    export DISPLAY=0.0




WORKDIR /u01/oracle/

ENV PATH $PATH:/u01/oracle/weblogic/wlserver_10.3/common/bin

# Define default command to start bash. 
CMD ["bash"]
