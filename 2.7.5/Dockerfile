FROM openjdk:8
LABEL maintainer="dengdi1803@gmail.com"

# init
RUN apt-get update && \
    apt-get -qy dist-upgrade &&\
    apt-get install -qy ssh pdsh vim curl && \
    mkdir -p /opt
WORKDIR /opt

# install hadoop
RUN curl -L https://archive.apache.org/dist/hadoop/core/hadoop-2.7.5/hadoop-2.7.5.tar.gz -o - | tar -xzf - && \
	mv hadoop-2.7.5 hadoop

# setup
WORKDIR /opt/hadoop
ENV PATH=/opt/hadoop/bin:/opt/hadoop/sbin:$JAVA_HOME/bin:$PATH
RUN sed --in-place='.ori' -e "s/\${JAVA_HOME}/\/usr\/local\/openjdk-8/" etc/hadoop/hadoop-env.sh

# Configure ssh client
RUN ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
    chmod 0600 ~/.ssh/authorized_keys && \
	echo "\nHost *\n" >> ~/.ssh/config && \
    echo "   StrictHostKeyChecking no\n" >> ~/.ssh/config && \
    echo "   UserKnownHostsFile=/dev/null\n" >> ~/.ssh/config

# Disable sshd authentication
RUN echo "root:root" | chpasswd  && \
    sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# Pseudo-Distributed Operation
COPY etc/core-site.xml etc/hadoop/core-site.xml
COPY etc/hdfs-site.xml etc/hadoop/hdfs-site.xml
RUN hdfs namenode -format


# SSH
EXPOSE 22
# hdfs://localhost:8020
EXPOSE 8020
# HDFS Web browser
EXPOSE 50070
# HDFS datanodes address
EXPOSE 50010
# HDFS datanodes ipc address
EXPOSE 50020
# HDFS datanodes http address
EXPOSE 50075
# HDFS secondary namenode http address
EXPOSE 50090

CMD service ssh start \
  && start-dfs.sh \
  && sleep infinity \
  && bash