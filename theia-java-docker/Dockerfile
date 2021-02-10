ARG version=latest
FROM zxytech/theia-ide:${version}

USER root

ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    LANG=C.UTF-8

RUN set -x; \
    apk add --no-cache \
    bash=5.1.0-r0 \
    bash-completion=2.11-r2 \
    git=2.30.1-r0 \
    tig=2.5.1-r0 \
    git-bash-completion=2.30.1-r0 \
    git-flow=1.12.3-r0 \
    git-lfs=2.13.1-r0 \
    openssl=1.1.1i-r0 \
    curl=7.74.0-r0 \
    unzip=6.0-r8 \
    openjdk8=8.272.10-r4 \
    maven=3.6.3-r0 \
    gradle=6.7.1-r0 \
    ; \
    apk add --no-cache --virtual .build-deps tzdata=2021a-r0; \
    echo "Asia/Shanghai" > /etc/timezone; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del --no-network .build-deps; \
    \
    export JAVA_HOME=$JAVA_HOME \
    LANG=$LANG;

USER theia
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]