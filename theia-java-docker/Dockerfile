ARG version=latest
FROM zxytech/theia-ide:${version}

USER root

ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    JAVA_VERSION=8u272 \ 
    JAVA_ALPINE_VERSION=8.272.10-r4 \ 
    LANG=C.UTF-8

RUN set -x; \
    apk add --no-cache openjdk8="$JAVA_ALPINE_VERSION"; \
    \
    apk add --no-cache --virtual .build-deps tzdata; \
    echo "Asia/Shanghai" > /etc/timezone; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del --no-network .build-deps; \
    \
    export JAVA_HOME=$JAVA_HOME \
    LANG=$LANG;

WORKDIR /home/project
USER theia
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]