ARG NODE_VERSION=erbium

FROM node:${NODE_VERSION}-alpine as builder

RUN apk add --no-cache make pkgconfig gcc g++ python libx11-dev libxkbfile-dev

ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN

RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean && \
    rm -rf .cache

FROM node:${NODE_VERSION}-alpine

ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    JAVA_VERSION=8u272 \ 
    JAVA_ALPINE_VERSION=8.272.10-r4 \ 
    LANG=C.UTF-8 \
    HOME=/home/theia \
    SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins \
    USE_LOCAL_GIT=true

# See : https://github.com/theia-ide/theia-apps/issues/34
RUN apk add --no-cache \
    git \
    git-flow \
    tig \
    bash \
    bash-completion \
    git-bash-completion \
    tree \
    curl \
    \
    openjdk8="$JAVA_ALPINE_VERSION" \
    maven \
    gradle \
    ; \
    apk add --no-cache --virtual .build-deps tzdata; \
    echo "Asia/Shanghai" > /etc/timezone; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del --no-network .build-deps; \
    \
    export JAVA_HOME=$JAVA_HOME \
    LANG=$LANG; \
    addgroup theia; \
    adduser -G theia -s /bin/bash -D theia; \
    \
    chmod g+rw /home; \
    mkdir -p /home/project; \
    chown -R theia:theia /home/theia; \
    chown -R theia:theia /home/project; \
    \
    echo "source /etc/profile" >> /home/theia/.bashrc;

WORKDIR /home/theia
COPY --from=builder --chown=theia:theia /home/theia /home/theia

EXPOSE 3000
USER theia
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]