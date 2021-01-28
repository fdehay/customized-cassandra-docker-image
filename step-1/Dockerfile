FROM clearlinux

RUN swupd bundle-add --quiet --skip-optional java-runtime wget diffutils which deprecated-python2

WORKDIR /opt

RUN useradd --home-dir /opt/cassandra cass

RUN wget -q 'https://ftp.wayne.edu/apache/cassandra/3.11.9/apache-cassandra-3.11.9-bin.tar.gz'
RUN wget -q 'https://downloads.apache.org/cassandra/3.11.9/apache-cassandra-3.11.9-bin.tar.gz.sha512'
RUN sha512sum apache-cassandra-3.11.9-bin.tar.gz | cut -d ' ' -f1 > apache-cassandra-3.11.9-bin.tar.gz.checksum
RUN diff -w apache-cassandra-3.11.9-bin.tar.gz.sha512 apache-cassandra-3.11.9-bin.tar.gz.checksum
RUN tar xzf apache-cassandra-3.11.9-bin.tar.gz -C cassandra --strip-components=1

USER cass

ENV CASSANDRA_HOME=/opt/cassandra
ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
ENV PATH=$PATH:$JAVA_HOME/bin:$CASSANDRA_HOME/bin

WORKDIR $CASSANDRA_HOME
