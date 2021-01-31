# Theia Docker Example


## Run the latest stable version

```
docker run -it -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia
```

Go to [http://localhost:3000](http://localhost:3000/) to access it.

## Run the nightly version

```
docker run -it -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia:next
```

Go to [http://localhost:3000](http://localhost:3000/) to access it.