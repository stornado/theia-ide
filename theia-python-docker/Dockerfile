ARG NODE_VERSION=erbium

FROM node:${NODE_VERSION}-alpine as builder

RUN apk add --no-cache make gcc g++ python

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
    yarn cache clean

FROM node:${NODE_VERSION}-alpine

# Install Python 3 from source
ARG VERSION=3.9.1

RUN apk update --no-cache \
    && apk upgrade --no-cache \
    && apk add --no-cache make build-essential libssl-dev \
    && apk add --no-cache libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    && apk add --no-cache libncurses5-dev  libncursesw5-dev xz-utils tk-dev \
    && wget https://www.python.org/ftp/python/$VERSION/Python-$VERSION.tgz \
    && tar xvf Python-$VERSION.tgz \
    && cd Python-$VERSION \
    && ./configure \
    && make -j8 \
    && make install \
    && cd .. \
    && rm -rf Python-$VERSION \
    && rm Python-$VERSION.tgz

RUN apk update --no-cache \
    && apk add --no-cache python-dev python-pip \
    && pip install --upgrade pip --user \
    && apk add --no-cache python3-dev python3-pip \
    && pip3 install --upgrade pip --user \
    && pip3 install python-language-server flake8 autopep8 \
    && apk add --no-cache yarn \
    && apt-get clean \
    && apt-get auto-remove -y \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

RUN mkdir -p /home/theia \
    && mkdir -p /home/project
ENV HOME /home/theia
WORKDIR /home/theia
COPY --from=builder /home/theia /home/theia
EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins
ENV USE_LOCAL_GIT true
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
