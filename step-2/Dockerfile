FROM clearlinux as tinisetup

WORKDIR /

RUN swupd bundle-add --quiet --skip-optional wget

ENV TINI_VERSION v0.19.0
RUN wget --no-check-certificate --no-cookies --quiet https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-amd64 \
    && wget --no-check-certificate --no-cookies --quiet https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-amd64.sha256sum \
    && echo "$(cat tini-amd64.sha256sum)" | sha256sum -c \
    && chmod +x /tini-amd64

FROM datastax/cassandra-mgmtapi-3_11_7:v0.1.14 AS mgmtapisetup

RUN ls /etc/cassandra/*.jar
RUN ls /opt/mgmtapi/*.jar

RUN rm /opt/mgmtapi/datastax-mgmtapi-shim-4.x-0.1.0-SNAPSHOT.jar

FROM clearlinux

RUN swupd bundle-add --quiet --skip-optional java-runtime wget diffutils which deprecated-python2

WORKDIR /opt

RUN useradd --home-dir /opt/cassandra cass

RUN wget -q 'https://ftp.wayne.edu/apache/cassandra/3.11.9/apache-cassandra-3.11.9-bin.tar.gz'
RUN wget -q 'https://downloads.apache.org/cassandra/3.11.9/apache-cassandra-3.11.9-bin.tar.gz.sha512'
RUN sha512sum apache-cassandra-3.11.9-bin.tar.gz | cut -d ' ' -f1 > apache-cassandra-3.11.9-bin.tar.gz.checksum
RUN diff -w apache-cassandra-3.11.9-bin.tar.gz.sha512 apache-cassandra-3.11.9-bin.tar.gz.checksum
RUN tar xzf apache-cassandra-3.11.9-bin.tar.gz -C cassandra --strip-components=1

COPY --from=mgmtapisetup /etc/cassandra/datastax-mgmtapi-*.jar /opt/cassandra/conf/

COPY --from=mgmtapisetup /opt/mgmtapi/datastax-mgmtapi-*.jar /opt/mgmtapi/

COPY ./docker-entrypoint.sh /
RUN chmod 755 /docker-entrypoint.sh

ENV MCAC_VERSION 0.1.12
RUN mkdir /opt/mcac-agent && wget -q -O /tmp/mcac-agent.tar.gz "https://github.com/datastax/metric-collector-for-apache-cassandra/releases/download/v${MCAC_VERSION}/datastax-mcac-agent-${MCAC_VERSION}.tar.gz" && tar xzf /tmp/mcac-agent.tar.gz -C /opt/mcac-agent --strip-components 1 && rm /tmp/mcac-agent.tar.gz

COPY --from=tinisetup /tini-amd64 /tini

RUN chown -R cass:cass /opt/cassandra/conf /opt/mcac-agent/config

RUN swupd bundle-add --quiet --skip-optional less htop

USER cass

ENV CASSANDRA_HOME=/opt/cassandra
ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
ENV PATH=$PATH:$JAVA_HOME/bin:$CASSANDRA_HOME/bin

WORKDIR $CASSANDRA_HOME
