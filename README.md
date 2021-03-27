# theia-ide

[Official Images](https://hub.docker.com/u/theiaide)

https://theia-ide.org/

## Theia Go Docker

Run on http://localhost:3000 with the current directory as a workspace:

```bash
docker run --security-opt seccomp=unconfined -e GO111MODULE=auto -it --init -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia-go:next
```

**Terminal xxx command not found**

```bash
source /etc/profile && source ~/.profile
```

## Theia Python Docker
 
Run locally on Linux or OS X

```bash
docker run -it --init -p 3000:3000 -v "$(pwd):/home/project" theiaide/theia-python:latest
```
