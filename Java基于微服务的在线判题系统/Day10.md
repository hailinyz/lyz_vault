
后端判断token是否过期，还有获取用户信息，头像昵称
获取用户信息接口
```java
/*  
 * 获取用户信息  
 */@Override  
public R<LoginUserVO> info(String token) {  
    if (StrUtil.isNotEmpty(token) &&  
            token.startsWith(HttpConstants.PREFIX)) {  
        token = token.replaceFirst(HttpConstants.PREFIX, StrUtil.EMPTY);  
    }  
    LoginUser loginUser = tokenService.getLoginUser(token, secret);  
    if (loginUser == null){  
        return R.fail();  
    }  
    LoginUserVO loginUserVO = new LoginUserVO();  
    loginUserVO.setNickName(loginUser.getNickName());  
    loginUserVO.setHeadImage(loginUser.getHeadImage());  
    return R.ok(loginUserVO);  
}
```
注意要新增参数，VO、实体类啥的还有调用的地方。


## C端竞赛列表页面

不同：
展示形式不同（前端处理）
每个竞赛展示的数据不同，C端的更少一些。（只需要调整查询sql）