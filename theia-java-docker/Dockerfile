ARG version=latest
FROM zxytech/theia-ide:${version}

USER root

ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    LANG=C.UTF-8

RUN set -x; \
    apk add --no-cache \
    bash \
    bash-completion \
    git \
    tig \
    git-bash-completion \
    # git-flow \
    git-lfs \
    openssl \
    curl \
    unzip \
    openjdk8 \
    maven \
    gradle \
    ; \
    apk add --no-cache --virtual .build-deps tzdata; \
    echo "Asia/Shanghai" > /etc/timezone; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del --no-network .build-deps; \
    \
    export JAVA_HOME=$JAVA_HOME \
    LANG=$LANG;

USER theia