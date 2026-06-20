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
