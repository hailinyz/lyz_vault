# 部署前准备
![](assets/Centos%207%20部署/file-20260620105816259.png)
重启网卡生效获取centos 内网ip
```powershell
systemctl restart network

# 查看ip
ip addr
```
![](assets/Centos%207%20部署/file-20260620110122474.png)

# 开启ssh服务
判断开没开启
```powershell
systemctl status sshd
```
![](assets/Centos%207%20部署/file-20260620110408664.png)
绿色已经安装

# 防火墙放行22端口（ssh默认端口）
先验证有没有放行
![](assets/Centos%207%20部署/file-20260620110638095.png)
no表示没有开
**开放**
```powershell
# 永久开放22端口
firewall-cmd --zone=public --add-port=22/tcp --permanent
# 重载防火墙生效
firewall-cmd --reload
# 验证是否开放成功
firewall-cmd --query-port=22/tcp
```
![](assets/Centos%207%20部署/file-20260620110823803.png)
yes表示开放成功

**然后就可以用MobaXterm_Personal_23.2.exe进行远程连接了**

# 接下来都用MobaXterm_Personal_23.2.exe进行操作

# 下载阿里云 CentOS7 国内镜像源
```powershell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

# 清理旧缓存、生成新缓存
```powershell
yum clean all 
yum makecache
```

# 安装 yum-utils
```powershell
yum install yum-utils -y
```

# 添加阿里云源
```powershell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

# 安装docker
```powershell
# 安装docker
yum install -y docker-ce docker-ce-cli containerd.io
# 验证docker
docker --version
```

# 修改配置打开远程访问docker，因为judge服务需要用到docker的代码沙箱
```powershell
vi /lib/systemd/system/docker.service 
找到ExecStart 开头的配置，注释原配置 进⾏备份 插⼊以下内容 
[Service] 
Type=notify 
# the default is not to use systemd for cgroups because the delegate issues still # exists and systemd currently does not support the cgroup feature set required 
# for containers run by docker 
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID 
TimeoutStartSec=0 
RestartSec=2 
Restart=always
```
![](assets/Centos%207%20部署/file-20260620121030534.png)

**到这里我们的项目部署的准备工作就已经完成了**

