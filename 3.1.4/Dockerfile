FROM openjdk:8
LABEL maintainer="dengdi1803@gmail.com"

# init
RUN apt-get update && \
    apt-get -qy dist-upgrade &&\
    apt-get install -qy ssh pdsh vim curl && \
    mkdir -p /opt
WORKDIR /opt

# install hadoop
RUN curl -L https://archive.apache.org/dist/hadoop/core/hadoop-3.1.4/hadoop-3.1.4.tar.gz -o - | tar -xzf - && \
	mv hadoop-3.1.4 hadoop

# setup
WORKDIR /opt/hadoop
ENV HDFS_NAMENODE_USER=root \
    HDFS_SECONDARYNAMENODE_USER=root \
	HDFS_DATANODE_USER=root \
    PATH=/opt/hadoop/bin:/opt/hadoop/sbin:$JAVA_HOME/bin:$PATH
RUN echo 'export JAVA_HOME=/usr/local/openjdk-8' >> etc/hadoop/hadoop-env.sh && \
    sed -i 's/if \[\[ -e '\''\/usr\/bin\/pdsh'\'' \]\]; then/if \[\[ ! -e '\''\/usr\/bin\/pdsh'\'' \]\]; then/g' libexec/hadoop-functions.sh && \
	sed -i 's/PDSH_SSH_ARGS_APPEND="${HADOOP_SSH_OPTS}" pdsh \`/PDSH_RCMD_TYPE=ssh PDSH_SSH_ARGS_APPEND="${HADOOP_SSH_OPTS}" pdsh \`/g' libexec/hadoop-functions.sh

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
# hdfs://localhost:9000
EXPOSE 9000
# HDFS Web browser
EXPOSE 9870
# HDFS datanodes address
EXPOSE 9866
# HDFS datanodes ipc address
EXPOSE 9867
# HDFS datanodes http address
EXPOSE 9864
# HDFS secondary namenode http address
EXPOSE 9868

CMD service ssh start \
  && start-dfs.sh \
  && sleep infinity \
  && bash