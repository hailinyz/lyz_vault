## 1.数据卷

**数据卷(volume)是一个虚拟目录，是容器内目录与宿主机目录之间映射的桥梁**
### 利用Nginx容器部署静态资源

| 命令                    | 说明         | 文档地址                                                                                                 |
| --------------------- | ---------- | ---------------------------------------------------------------------------------------------------- |
| docker volume create  | 创建数据卷      | [docker volume create \| Docker Docs](https://docs.docker.com/reference/cli/docker/volume/create/)   |
| docker volume ls      | 查看所有数据卷    | [docker volume ls \| Docker Docs](https://docs.docker.com/reference/cli/docker/volume/ls/)           |
| docker volume rm      | 删除指定数据卷    | [docker volume rm \| Docker Docs](https://docs.docker.com/reference/cli/docker/volume/rm/)           |
| docker volume inspect | 查看某个数据卷的详情 | [docker volume inspect \| Docker Docs](https://docs.docker.com/reference/cli/docker/volume/inspect/) |
| docker volume prune   | 清除数据卷      | [docker volume prune \| Docker Docs](https://docs.docker.com/reference/cli/docker/volume/prune/)     |
