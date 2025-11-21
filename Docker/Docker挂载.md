## 1.数据卷挂载

**数据卷(volume)是一个虚拟目录，是容器内目录与宿主机目录之间映射的桥梁**
### 利用Nginx容器部署静态资源

| 命令                    | 说明         | 文档地址                                                                                  |
| --------------------- | ---------- | ------------------------------------------------------------------------------------- |
| docker volume create  | 创建数据卷      | [docker volume create](https://docs.docker.com/reference/cli/docker/volume/create/)   |
| docker volume ls      | 查看所有数据卷    | [docker volume ls ](https://docs.docker.com/reference/cli/docker/volume/ls/)          |
| docker volume rm      | 删除指定数据卷    | [docker volume rm ](https://docs.docker.com/reference/cli/docker/volume/rm/)          |
| docker volume inspect | 查看某个数据卷的详情 | [docker volume inspect](https://docs.docker.com/reference/cli/docker/volume/inspect/) |
| docker volume prune   | 清除数据卷      | [docker volume prune](https://docs.docker.com/reference/cli/docker/volume/prune/)     |
+ 创建完宿主机目录之后
+ 挂载容器目录(做挂载就会自动创建了)
```powershell
docker run -v 数据卷:容器内目录
```
**所以在宿主机内做的所有操作容器内也实现了，这是利用数据卷实现了宿主机目录与容器内目录自动的双向的映射**

## 2.本地目录挂载

```powershell
docker run -v 本地目录:容器内目录
```

> [!NOTE] 注意
> -v mysql:/var/lib/mysql 会被识别为一个数据卷叫mysql
> -v ./mysql:/var/lib/mysql 会被识别为当前目录下的mysql目录
