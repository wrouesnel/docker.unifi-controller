ARG DOCKER_PREFIX=

FROM ${DOCKER_PREFIX}ubuntu:bionic

ARG URL_P2CLI=https://github.com/wrouesnel/p2cli/releases/download/r14/p2-linux-x86_64
ARG URL_MONGODB_TRUSTED_KEY=https://www.mongodb.org/static/pgp/server-3.4.asc
ARG URL_UNIFI_TRUSTED_KEY=https://dl.ui.com/unifi/unifi-repo.gpg

ARG http_proxy=
ARG https_proxy=

COPY tree-preinstall/ /

RUN apt update && apt install --no-install-recommends -y gnupg ca-certificates apt-transport-https wget curl

RUN \
    wget -O - /etc/apt/trusted.gpg.d/mongodb-repo.gpg "$URL_MONGODB_TRUSTED_KEY" | apt-key add - \
 && wget -O - /etc/apt/trusted.gpg.d/unifi-repo.gpg "$URL_UNIFI_TRUSTED_KEY" | apt-key add - \
 && echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" >> /etc/apt/sources.list.d/mongodb-org-3.4.list \
 && echo 'deb https://www.ui.com/downloads/unifi/debian stable ubiquiti' | tee /etc/apt/sources.list.d/100-ubnt-unifi.list

ENV \
  JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 \
  JVM_MAX_HEAP_SIZE=1024M \
  JVM_INIT_HEAP_SIZE= \
  DATA_DIR=/data

RUN apt update && apt install --no-install-recommends -y \
        openjdk-11-jre \
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
    # Templating engine
    curl -L "$URL_P2CLI" --output /usr/local/bin/p2 && \
    chmod +x /usr/local/bin/p2 && \
    apt install -y unifi && \
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

EXPOSE 6789/tcp 8080/tcp 8443/tcp 8880/tcp 8843/tcp 3478/udp

COPY healthcheck.sh /healthcheck.sh
HEALTHCHECK --interval=10s --timeout=10s --retries=3 CMD /healthcheck.sh

ENTRYPOINT [ "/entrypoint.bsh" ]
