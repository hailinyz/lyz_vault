## 个人中心 

### 查看用户详情 & 用户基本信息编辑

目前有个问题，就是C端用户登录/注册的时候，前端存不下token令牌，但是redis能存入token。
目前先这么将就着吧，可以先不通过网关。（理论上获取用户信息和用户详细信息都是要走到网关才行的）

###### !!!!!!!!!!!!!重磅，经过我的不断调试网关的过滤以及统一了nacos上的jwt配置
![](assets/Day13/file-20260109210634689.png)
注意，之前的nacos不统一，是这样的：
```cao
system：sdfghuijasxdjkawskuigy
friend：sdfghuijasxdjkawskuigysms
gateway：sdfghuijasxdjkawskuigy
job：sdfghuijasxdjkawskuigy
```

统一后才对，因为生成token和解析token的jwt应该是一样的才能正确解析出userId
```cao
system：sdfghuijasxdjkawskuigy
friend：sdfghuijasxdjkawskuigy
gateway：sdfghuijasxdjkawskuigy
job：sdfghuijasxdjkawskuigy
```