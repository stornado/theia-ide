ARG NODE_VERSION=erbium

FROM node:${NODE_VERSION} as builder

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

FROM node:${NODE_VERSION}

# Install Python 3 from source
ARG VERSION=3.9.1

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y make build-essential libssl-dev \
    && apt-get install -y libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    && apt-get install -y libncurses5-dev  libncursesw5-dev xz-utils tk-dev \
    && wget https://www.python.org/ftp/python/$VERSION/Python-$VERSION.tgz \
    && tar xvf Python-$VERSION.tgz \
    && cd Python-$VERSION \
    && ./configure \
    && make -j8 \
    && make install \
    && cd .. \
    && rm -rf Python-$VERSION \
    && rm Python-$VERSION.tgz \
    \
    && apt-get update \
    && apt-get install -y python-dev python-pip \
    && pip install --upgrade pip --user \
    && apt-get install -y python3-dev python3-pip \
    && pip3 install --upgrade pip --user \
    && pip3 install python-language-server flake8 autopep8 \
    && apt-get install -y yarn \
    && apt-get clean \
    && apt-get auto-remove -y \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* \
    \
    && mkdir -p /home/theia \
    && mkdir -p /home/project \
    \
    && echo "export PATH=\$PATH:/bin:/sbin" >> /home/theia/.bashrc \
    && echo "source /etc/profile" >> /home/theia/.bashrc

WORKDIR /home/theia
COPY --from=builder /home/theia /home/theia

EXPOSE 3000

ENV SHELL=/bin/bash \
    HOME=/home/theia \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins \
    USE_LOCAL_GIT=true

ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
