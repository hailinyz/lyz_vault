
用户每次提交代码都要经过停止容器->删除容器，就很慢

面临着多线程一样的问题，new之后还要删除，搞了个线程池；同样的，我们也可以有容器池。

我们的DockerClient有了，后续我们使用DockerSandBox是从容器池里面拿这个容器去使用的

阻塞队列

代码的改动：把之前那种每提交一次代码都要创建一个新容器**改成**从容器池中直接拿容器的方式；清理不在是删除容器而是**归还**的方式。

短时间内可能会有大量的请求，导致崩溃（就该用rabbitmq的**流量削峰**）


拉取镜像
```powershell
docker pull rabbitmq:3.8.30-management
```
启动容器
```powershell
docker run -d --name oj-rabbit-dev -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:3.8.30-management
```
进入容器内部并启动管理插件
![](assets/Day16/file-20260115141533507.png)
```powershell
rabbitmq-plugins enable rabbitmq_management
```


创建oj-common-rabbitmq⼯程
