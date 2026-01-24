
## 项目部署 - Nginx

Nginx是一个HTTP和反向代理Web服务器。有 **反向代理、负载均衡、动静分离**等功能。

### 正向代理
![](assets/Day%2019/file-20260124200756816.jpeg)

### 反向代理
![](assets/Day%2019/file-20260124200959867.jpeg)

### 负载均衡

轮询、权重、哈希（每个访客固定访问一个后端服务器）、最少链接（新连接分配到连接数少的服务器）

### 动静分离

![](assets/Day%2019/file-20260124201747091.png)
![](assets/Day%2019/file-20260124201911388.jpeg)

为什么在部署的时候才引入？因为本地开发通常无需nginx。专注于业务功能的实现。
一旦开发完成，部署到服务器上，就要考虑性能、安全所以需要。

---


### 安装

拉取镜像
```powershell
docker pull nginx:1.21
```
启动nginx容器
```powershell
docker run --name oj-nginx -d -p 5413:80 nginx:1.21
```
启动成功
![](assets/Day%2019/file-20260124202355815.png)



