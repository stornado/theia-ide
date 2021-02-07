ARG NODE_VERSION=erbium

FROM node:$NODE_VERSION-alpine as theia

RUN apk add --no-cache make gcc g++ python

ARG GITHUB_TOKEN
ARG version=latest

WORKDIR /home/theia

ADD $version.package.json ./package.json
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

FROM node:$NODE_VERSION-alpine

COPY --from=theia /home/theia /home/theia

WORKDIR /home/theia


ENV GOLANG_VERSION=1.15.8 \
    GO_VERSION=1.15 \
    \
    GOROOT=/home/go \
    GOPATH=/home/go-tools \
    \
    PATH=$GOPATH/bin:$GOROOT/bin:$PATH

RUN apk add --no-cache \
    ca-certificates \
    bash \
    openssh \
    git; \
    \
    # See: https://github.com/theia-ide/theia-apps/issues/34
    adduser --disabled-password --gecos '' theia; \
    chmod g+rw /home; \
    mkdir -p /home/project; \
    mkdir -p /home/go; \
    mkdir -p /home/go-tools; \
    echo "export PATH=/home/project/bin:/home/go-tools/bin:/home/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" >> /home/theia/.profile; \
    echo "source /etc/profile" >> /home/theia/.bashrc; \
    echo "source /home/theia/.profile" >> /home/theia/.bashrc; \
    \
    chown -R theia:theia /home/theia; \
    chown -R theia:theia /home/project; \
    chown -R theia:theia /home/go; \
    chown -R theia:theia /home/go-tools; \
    \
    # set up nsswitch.conf for Go's "netgo" implementation
    # - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
    # - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
    [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf; \
    \
    set -eux; \
    apk add --no-cache --virtual .build-deps \
    bash \
    gcc \
    gnupg \
    go \
    musl-dev \
    openssl \
    ; \
    export \
    # set GOROOT_BOOTSTRAP such that we can actually build Go
    GOROOT_BOOTSTRAP=/home/go \
    # ... and set "cross-building" related vars to the installed system's values so that we create a build targeting the proper arch
    # (for example, if our build host is GOARCH=amd64, but our build env/image is GOARCH=386, our build needs GOARCH=386)
    GOOS="$(go env GOOS)" \
    GOARCH="$(go env GOARCH)" \
    GOHOSTOS="$(go env GOHOSTOS)" \
    GOHOSTARCH="$(go env GOHOSTARCH)" \
    \
    GO_VERSION=1.15 \
    GOROOT=/home/go \
    GOPATH=/home/go-tools \
    ; \
    echo "export PATH=/home/go-tools/bin:/home/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" >> /etc/profile; \
    export PATH=$GOPATH/bin:$GOROOT/bin:$PATH \
    ; \
    # also explicitly set GO386 and GOARM if appropriate
    # https://github.com/docker-library/golang/issues/184
    apkArch="$(apk --print-arch)"; \
    case "$apkArch" in \
    armhf) export GOARM='6' ;; \
    armv7) export GOARM='7' ;; \
    x86) export GO386='387' ;; \
    esac; \
    \
    # https://github.com/golang/go/issues/38536#issuecomment-616897960
    url='https://storage.googleapis.com/golang/go1.15.8.src.tar.gz'; \
    sha256='540c0ab7781084d124991321ed1458e479982de94454a98afab6acadf38497c2'; \
    \
    wget -O go.tgz.asc "$url.asc"; \
    wget -O go.tgz "$url"; \
    echo "$sha256 *go.tgz" | sha256sum -c -; \
    \
    # https://github.com/golang/go/issues/14739#issuecomment-324767697
    export GNUPGHOME="$(mktemp -d)"; \
    # https://www.google.com/linuxrepositories/
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys 'EB4C 1BFD 4F04 2F6D DDCC EC91 7721 F63B D38B 4796'; \
    gpg --batch --verify go.tgz.asc go.tgz; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME" go.tgz.asc; \
    \
    tar -C /home -xzf go.tgz; \
    rm go.tgz; \
    \
    goEnv="$(go env | sed -rn -e '/^GO(OS|ARCH|ARM|386)=/s//export \0/p')"; \
    eval "$goEnv"; \
    [ -n "$GOOS" ]; \
    [ -n "$GOARCH" ]; \
    ( \
    cd /home/go/src; \
    ./make.bash; \
    ); \
    \
    # pre-compile the standard library, just like the official binary release tarballs do
    go install std; \
    # go install: -race is only supported on linux/amd64, linux/ppc64le, linux/arm64, freebsd/amd64, netbsd/amd64, darwin/amd64 and windows/amd64
    #	go install -race std; \
    \
    # Install VS Code Go tools: https://github.com/Microsoft/vscode-go/blob/058eccf17f1b0eebd607581591828531d768b98e/src/goInstallTools.ts#L19-L45
    go get -u -v github.com/mdempsky/gocode; \
    go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs; \
    go get -u -v github.com/ramya-rao-a/go-outline; \
    go get -u -v github.com/acroca/go-symbols; \
    go get -u -v golang.org/x/tools/cmd/guru; \
    go get -u -v golang.org/x/tools/cmd/gorename; \
    go get -u -v github.com/fatih/gomodifytags; \
    go get -u -v github.com/haya14busa/goplay/cmd/goplay; \
    go get -u -v github.com/josharian/impl; \
    go get -u -v github.com/tylerb/gotype-live; \
    go get -u -v github.com/rogpeppe/godef; \
    go get -u -v github.com/zmb3/gogetdoc; \
    go get -u -v golang.org/x/tools/cmd/goimports; \
    go get -u -v github.com/sqs/goreturns; \
    go get -u -v winterdrache.de/goformat/goformat; \
    go get -u -v golang.org/x/lint/golint; \
    go get -u -v github.com/cweill/gotests/...; \
    go get -u -v github.com/alecthomas/gometalinter; \
    go get -u -v honnef.co/go/tools/...; \
    GO111MODULE=on go get github.com/golangci/golangci-lint/cmd/golangci-lint; \
    go get -u -v github.com/mgechev/revive; \
    go get -u -v github.com/sourcegraph/go-langserver; \
    go get -u -v github.com/go-delve/delve/cmd/dlv; \
    go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct; \
    go get -u -v github.com/godoctor/godoctor; \
    \
    go get -u -v -d github.com/stamblerre/gocode; \
    go build -o $GOPATH/bin/gocode-gomod github.com/stamblerre/gocode; \
    \
    GO111MODULE=on go get -u -v golang.org/x/tools/gopls; \
    \
    apk del --no-network .build-deps; \
    \
    # remove a few intermediate / bootstrapping files the official binary release tarballs do not contain
    rm -rf \
    /home/go/pkg/*/cmd \
    /home/go/pkg/bootstrap \
    /home/go/pkg/obj \
    /home/go/pkg/tool/*/api \
    /home/go/pkg/tool/*/go_bootstrap \
    /home/go/src/cmd/dist/dist \
    \
    /home/theia/.cache \
    ; \
    go version

USER theia

# Configure Theia
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins  \
    # Configure user Go path
    GOPATH=/home/project \
    \
    PATH=$GOPATH/bin:$PATH \
    USE_LOCAL_GIT=true

EXPOSE 3000
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
