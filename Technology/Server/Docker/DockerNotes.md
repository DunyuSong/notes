# Docker Notes

## Docker 命令

- 运行的images列表：  docker ps
- 进入docker：  docker exec -it containerID /bash/sh
- 从宿主机拷贝文件到容器：docker cp  /opt/test/ containerID:/opt/testnew/file.txt
- 查看docker日志：docker logs -f -t --tail 20 容器ID