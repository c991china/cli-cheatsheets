# Docker 速查

## 镜像 / 容器
    docker images
    docker ps -a
    docker run -d -p 8080:80 --name web nginx
    docker rm -f <id>
    docker rmi <img>

## 构建
    docker build -t myapp:1.0 .
    docker tag myapp:1.0 reg.io/myapp:1.0
    docker push reg.io/myapp:1.0

## 清理
    docker system prune -a --volumes
    docker volume ls

## 进入容器
    docker exec -it <id> sh
