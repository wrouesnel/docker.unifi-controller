# Standalone PowerDNS recursor
ARG DOCKER_PREFIX=

FROM ${DOCKER_PREFIX}ubuntu:zesty

MAINTAINER Will Rouesnel <w.rouesnel@gmail.com>

ARG URL_P2CLI=https://github.com/wrouesnel/p2cli/releases/download/r4/p2
#ARG URL_NGINX_EXPORTER=https://github.com/wrouesnel/nginx_exporter/releases/download/v0.0.1/nginx_exporter
#ARG URL_PDNS_EXPORTER=https://github.com/wrouesnel/pdns_exporter/releases/download/v0.0.3/pdns_exporter.x86_64
#ARG URL_GOSU=https://github.com/tianon/gosu/releases/download/1.10/gosu-amd64
#ARG URL_PDNS_REPO=http://repo.powerdns.com/ubuntu

ARG URL_UNIFI=http://dl.ubnt.com/unifi/5.4.16/unifi_sysvinit_all.deb

ARG http_proxy=
ARG https_proxy=

COPY tree-preinstall/ /

COPY *.asc /tmp/

RUN apt-key add /tmp/*.asc && rm -f /tmp/*.asc

ENV \
  JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 \
  JVM_MAX_HEAP_SIZE=1024M \
  JVM_INIT_HEAP_SIZE= \
  DATA_DIR=/data

RUN apt-get update && apt-get install --no-install-recommends -y \
        openjdk-8-jre \
        mongodb-server \
        binutils \
        jsvc \
        ca-certificates ca-certificates-java \
        runit \
        syslog-ng-core \
        openssl \
        pwgen \
        openssl \
        pwgen \
        sudo \
        curl dnsutils apache2-utils && \
    # gosu
    #wget -O /usr/local/bin/gosu $URL_GOSU && \
    #chmod +x /usr/local/bin/gosu && \
    # Templating engine
    curl -L $URL_P2CLI --output /usr/local/bin/p2 && \
    chmod +x /usr/local/bin/p2 && \
    # Prometheus monitoring for nginx
    #wget -O /usr/local/bin/nginx_exporter $URL_NGINX_EXPORTER && \
    #chmod +x /usr/local/bin/nginx_exporter && \
    # Prometheus monitoring for powerdns
    #wget -O /usr/local/bin/pdns_exporter $URL_PDNS_EXPORTER && \
    #chmod +x /usr/local/bin/pdns_exporter && \
    curl -L $URL_UNIFI --output /tmp/unifi_sysvinit_all.deb && \
    dpkg -i /tmp/unifi_sysvinit_all.deb && \
    rm -f /tmp/unifi_sysvinit_all.deb && \
    ln -sf ${DATA_DIR}/unifi-controller/data /usr/lib/unifi/data  && \
    ln -sf /run/unifi-logs /usr/lib/unifi/logs  && \
    ln -sf ${DATA_DIR}/unifi-controller/work /usr/lib/unifi/work  && \
    ln -sf /run/unifi /usr/lib/unifi/run  && \
    # Cleanup    
    apt-get autoremove -y
    # Delete nginx default configuration.
    #rm -rf /etc/nginx/sites-enabled /etc/nginx/sites-available /etc/nginx/snippets /etc/nginx/conf.d

COPY tree-postinstall/ /

ENV \
    DEV_ALLOW_EPHEMERAL_DATA=no

#ENV API_KEY= \
#    API_ADMIN_USER=admin \
#    API_ADMIN_PASSWORD= \
#    API_AUTH=yes \
#    CORE_COUNT=1 \
#    POWERDNS_DATA=/data \
#    SSL_SERVER_CERT= \
#    SSL_SERVER_CERTCHAIN= \
#    SSL_SERVER_KEY= \
#    DEV_ALLOW_SELF_SIGNED=no \
#    DEV_ALLOW_EPHEMERAL_DATA=no \
#    DEV_ENABLE_TRACE=no

EXPOSE 6789/tcp 8080/tcp 8443/tcp 8880/tcp 8843/tcp 3478/udp

#EXPOSE 443/tcp 53/udp 53/tcp

COPY healthcheck.sh /healthcheck.sh
HEALTHCHECK --interval=10s --timeout=10s --retries=3 CMD /healthcheck.sh

ENTRYPOINT [ "/entrypoint.bsh" ]
