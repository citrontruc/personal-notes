# Docker

docker image prune to remove images with no containers attached to them.
docker container prune.

To remove select images:

```bash
docker rmi Image Image
```

docker ps -a to see all docker containers (even stopped ones)

Access to a docker container with a bash:

```bash
docker run -it --rm mcr.microsoft.com/dotnet/sdk:9.0 bash
```

Build container and give it a name:

```bash
docker build -t todoapi .
```

Run container with port option:

```bash
docker run -p 7111:7111 todoapi
```

And with docker compose.

```bash
docker compose up # build + run (add - between the two on windows)
docker compose build
```
