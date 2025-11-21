
**自定义镜像就是包含了==应用程序、程序运行的系统函数库、运行配置等文件==的文件包。构建镜像的过程其实就是把上述文件==打包==的过程**

镜像是分层的，有入口、出口，还有一些配置的中间层

## 网络

假如自定义网络的容器才能通过容器名互相访问，Docker的网络操作命令如下

| 命令                          | 说明           | 文档地址                                                                                                  |
| --------------------------- | ------------ | ----------------------------------------------------------------------------------------------------- |
| `docker network create`     | 创建一个网络       | [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/)         |
| `docker network ls`         | 查看所有网络       | [docker network ls](https://docs.docker.com/engine/reference/commandline/network_ls/)                 |
| `docker network rm`         | 删除指定网络       | [docker network rm](https://docs.docker.com/engine/reference/commandline/network_rm/)                 |
| `docker network prune`      | 清除未使用的网络     | [docker network prune](https://docs.docker.com/engine/reference/commandline/network_prune/)           |
| `docker network connect`    | 使指定容器连接加入某网络 | [docker network connect](https://docs.docker.com/engine/reference/commandline/network_connect/)       |
| `docker network disconnect` | 使指定容器连接离开某网络 | [docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/) |
| `docker network inspect`    | 查看网络详细信息     | [docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/)       |
## 部署后端项目

## 部署前端项目

## 注意：MYSQL、后端、前端都要在同一个网段内

以上手动部署太过复杂，没有整体性没有管理性，所以有了：+++++++++

## 11111.DockerComposeyi'jian部署   

