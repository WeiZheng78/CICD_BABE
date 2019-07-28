FROM ubuntu:16.04

ENV container docker

# Don't start any optional services except for the few we need.
RUN find /etc/systemd/system \
    /lib/systemd/system \
    -path '*.wants/*' \
    -not -name '*journald*' \
    -not -name '*systemd-tmpfiles*' \
    -not -name '*systemd-user-sessions*' \
    -exec rm \{} \;

RUN apt-get update && \
    apt-get install -y \
    dbus && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN systemctl set-default multi-user.target

COPY setup /sbin/

STOPSIGNAL SIGRTMIN+3

# RUN apt-get update && apt-get -y install sudo git curl wget python openssh-server nodejs openssh-client software-properties-common maven nodejs apt-transport-https ca-certificates gnupg2 vim software-properties-common

#RUN curl -O https://bootstrap.pypa.io/get-pip.py  && \
#    python get-pip.py && \
#    pip install ansible --upgrade

#RUN wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
#RUN echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000
ARG JENKINS_HOME=/var/lib/jenkins

ENV JENKINS_HOME $JENKINS_HOME
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}

# Jenkins is run with user `jenkins`, uid = 1000
# # If you bind mount a volume from the host or a data container,
# # ensure you use the same uid
RUN mkdir -p $JENKINS_HOME \
    && chown ${uid}:${gid} $JENKINS_HOME \
    && groupadd -g ${gid} ${group} \
    && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user} -p 1234 

ARG user=ansadm
ARG group=ansadm
ARG uid=1001
ARG gid=1001
ARG ANSADM_HOME=/home/ansadm
RUN mkdir -p $ANSADM_HOME \
    && chown ${uid}:${gid} $ANSADM_HOME \
    && groupadd -g ${gid} ${group} \
    && useradd -d "$ANSADM_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user} -p 1234

# for main web interface:
EXPOSE ${http_port}
#
# will be used by attached slave agents:
EXPOSE ${agent_port}
# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
# VOLUME $JENKINS_HOME

#RUN curl -O https://bootstrap.pypa.io/get-pip.py  && \
#    python get-pip.py && \
#    pip install ansible --upgrade

RUN apt-get update && \
    apt-get -y install apt-transport-https software-properties-common

RUN add-apt-repository ppa:openjdk-r/ppa && \
    apt-get update && \
    apt-get -y install telnet expect lsof iputils-ping openjdk-8-jdk sudo git curl wget python openssh-server nodejs openssh-client software-properties-common maven nodejs ca-certificates gnupg2 vim ansible && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" && \
    apt-get update && \
    apt-get -y install docker-ce

RUN curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose

RUN wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add - && \
    echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list

RUN wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb && \
    dpkg -i puppetlabs-release-pc1-xenial.deb && \
    apt-get update -y && apt-get -y install puppetserver jenkins && \
    /opt/puppetlabs/puppet/bin/puppet module install puppetlabs-docker

RUN systemctl enable puppetserver

RUN chmod +w /etc/sudoers && \
    echo "jenkins ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    echo "ansadm ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    chmod -w /etc/sudoers && \
    usermod -aG docker jenkins && usermod -aG docker ansadm

#RUN curl -O https://bootstrap.pypa.io/get-pip.py  && \
#    python get-pip.py && \
#    pip install ansible --upgrade

# Workaround for docker/docker#27202, technique based on comments from docker/docker#9212
CMD ["/bin/bash", "-c", "exec /sbin/init --log-target=journal 3>&1"]
