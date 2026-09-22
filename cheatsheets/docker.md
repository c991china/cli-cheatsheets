# docker cheatsheet

Docker 25 / Docker Desktop 4.28. Compose v2 (`docker compose`, no hyphen).

## Images

```bash
docker images                        # list
docker pull nginx:1.25-alpine
docker build -t myapp:dev .
docker build --no-cache -t myapp .   # ignore cache
docker build --progress=plain .      # full build log
docker tag myapp:dev registry.io/team/myapp:dev
docker push registry.io/team/myapp:dev
docker rmi myapp:dev                 # remove (fails if container uses it)
docker rmi -f myapp:dev              # force (destructive)
docker history myapp:dev             # layer sizes, find the fat layer
docker save myapp:dev | gzip > myapp.tgz   # export
docker load < myapp.tgz
```

## Run

```bash
docker run -d --name web -p 8080:80 nginx:1.25
docker run --rm -it ubuntu:22.04 bash       # throwaway interactive
docker run -e FOO=bar -e BAZ=1 myapp
docker run -v $(pwd):/app -w /app myapp python -m pytest
docker run -v pgdata:/var/lib/postgresql/data postgres:16
docker run --network mynet --name web myapp
docker run --memory 512m --cpus 1.5 myapp
docker run --restart unless-stopped -d myapp
docker run --entrypoint sh myapp -c 'ls -la'   # override entrypoint
```

## Containers

```bash
docker ps                            # running
docker ps -a                         # including stopped
docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker start|stop|restart <c>
docker rm <c>                        # remove stopped
docker rm -f <c>                     # force remove running (destructive)
docker rename old new
docker pause <c> / docker unpause <c>
```

## Inspect / debug

```bash
docker logs -f --tail 100 <c>
docker logs --since 10m <c>
docker exec -it <c> bash             # or sh on slim images
docker exec -u root -it <c> sh       # as root
docker inspect <c> | jq '.[0].State'
docker inspect <c> --format '{{.State.OOMKilled}}'
docker stats                         # live cpu/mem per container
docker top <c>                       # processes inside
docker port <c>                      # port mappings
docker cp <c>:/app/log.txt ./         # copy out
docker cp ./conf <c>:/etc/conf        # copy in
docker diff <c>                       # files changed in the container layer
docker events                         # live daemon events
```

## Cleanup (mind the destructive ones)

```bash
docker system df                     # how much space each category uses
docker system df -v                  # per-object breakdown
docker container prune               # remove stopped containers
docker image prune                   # dangling images only
docker image prune -a                # ALL unused images (destructive)
docker volume prune                  # unused volumes (destructive: data loss!)
docker builder prune                 # build cache
docker system prune                  # stopped containers, networks, dangling images
docker system prune -a --volumes     # nuke everything unused (destructive)
```

## Volumes

```bash
docker volume ls
docker volume create pgdata
docker volume inspect pgdata
docker volume rm pgdata              # fails if in use
docker run --rm -v pgdata:/data alpine ls -la /data
```

## Networks

```bash
docker network ls
docker network create mynet
docker network inspect mynet
docker network connect mynet <c>
docker network disconnect mynet <c>
```

Containers on the same user-defined network resolve each other by name. The
default `bridge` network does not.

## Compose

```bash
docker compose up -d                 # start detached
docker compose up -d --build         # rebuild images first
docker compose down                  # stop + remove containers/networks
docker compose down -v               # also remove volumes (destructive)
docker compose ps
docker compose logs -f web
docker compose exec web sh
docker compose run --rm web python manage.py migrate
docker compose config                # render the resolved YAML
docker compose pull                  # update images
```

## Registry / auth

```bash
docker login registry.io
docker logout
echo "$TOKEN" | docker login -u user --password-stdin registry.io
```

## Notes

- Exit code 137 = SIGKILL, usually OOM. Check `OOMKilled` in inspect.
- `--rm` auto-removes on exit; don't use it if you want the logs after.
- Build cache lives in `docker builder prune` territory and can be tens of GB.
