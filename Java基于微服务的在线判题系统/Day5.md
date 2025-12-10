## Entity、DTO、VO

Entity：实体类
DTO：后端接收前端请求参数，通常只包含需要返回的数据
VO：后端返回给前端的数据

解耦，修改每一层的时候对其他层的影响降低
合理利用网络资源，在互相传输数据过程中，尽可能用到哪些封装哪些

如果划分太细可能未来维护困难

## 对参数的校验

引入依赖validation
```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

这几个注解
```java
@Getter  
@Setter  
public class ValidationDTO {  
    @NotNull(message = "昵称不能为null")  
    private String nickName;  
  
    @NotBlank(message = "用户账号不能为空")  
    private String userAccount;  
  
    @NotEmpty(message = "用户密码不能为空，并且不能只包含空格")  
    @Size(min = 5, max = 10, message = "密码长度不能少于5位，不能大于10位")  
    private String password;  
  
    @Min(value = 0, message = "年龄不能小于0岁")  
    @Max(value = 60, message = "年龄不能大于60岁")  
    private int age;  
  
    @Email(message = "必须符合邮箱格式")  
    private String email;  
  
    @Pattern(regexp = "^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\\d{8}$",message = "手机号码格式不正确")  
    private String phone;  
  
    @Past(message = "开始日期必须是过去的日期")  
    private LocalDate startDate;  
  
    @Future(message = "结束日期必须是未来的日期")  
    private LocalDate endDate;  
}
```

注意加上这个配置才会对规则进行合法性判断
```java
@GetMapping("/validation")  
public String validation(@Validated ValidationDTO validationDTO){  
    return "参数测试";  
}
```

在全局异常处理加上具体参数失败处理方法，让其DTO配置的信息显示出来
```java
@ExceptionHandler(BindException.class)  
public R<Void> handleBindException(BindException e) {  
    log.error(e.getMessage());  
    String message = join(e.getAllErrors(),  
            DefaultMessageSourceResolvable::getDefaultMessage, ", ");  
    return R.fail(ResultCode.FAILED_PARAMS_VALIDATE.getCode(), message);  
}  
private <E> String join(Collection<E> collection, Function<E, String>  
        function, CharSequence delimiter) {  
    if (CollUtil.isEmpty(collection)) {  
        return StrUtil.EMPTY;  
    }  
    return collection.stream().map(function).filter(Objects::nonNull).collect(Collectors.joining(delimiter));  
}
```


## 前端项目创建oj-fe-b

安装element-plus
```powershell
npm install element-plus --save
```

按需导入element-plus，减小项目体积
```powershell
npm install -D unplugin-vue-components unplugin-auto-import
```

```js
import { fileURLToPath, URL } from 'node:url'

  

import vue from '@vitejs/plugin-vue'

import vueDevTools from 'vite-plugin-vue-devtools'

import { defineConfig } from 'vite'

import AutoImport from 'unplugin-auto-import/vite'

import Components from 'unplugin-vue-components/vite'

import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

  

// https://vite.dev/config/

export default defineConfig({

  plugins: [

    vue(),

    vueDevTools(),

        AutoImport({

      resolvers: [ElementPlusResolver()],

    }),

    Components({

      resolvers: [ElementPlusResolver()],

    }),

  ],

  resolve: {

    alias: {

      '@': fileURLToPath(new URL('./src', import.meta.url))

    },

  },

})
```

**登录成功**
携带用户信息向后端发请求-->后端接收前端请求，执行登录相关业务逻辑-->后端将结果返回给前端
如果登录成功，跳转到后台管理页面，并且前端将返回的token存储起来

**登陆失败**
提示一下失败信息

双向数据绑定v-model="username"

```vue
<el-input v-model="username" placeholder="请输⼊账号" />
```

```vue
<script setup>
import { ref } from 'vue'
const username = ref('')
const password = ref('')
</script>
```