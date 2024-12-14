
作为后端开发，进行Web Api 调试，除了使用 Postman, Apifox 等 Web Api 调试工具之外，我想使用Swagger进行调试应该是更方便，更常用的方式了吧。


![](https://img2024.cnblogs.com/blog/665705/202412/665705-20241213155549031-1395430637.png)


![](https://img2024.cnblogs.com/blog/665705/202412/665705-20241213155604901-165478934.png)


那么在需要 token 授权的情况下，每次调试都需要调用一下登录API, 然后复制返回的 token, 再粘贴到上图的文本框里，点一下Authorize按钮，是不是很麻烦呢？


**我的懒病犯了，我真的不想再复制粘贴token啦\~**


要是可以只要调用一下登录接口，返回的token就自动添加进去，自动授权，那该多好啊\~


**Let's go!**


##### 第一步：添加 custom.js 文件


1. 在 webapi 的项目中添加`wwwroot`文件夹；
2. 新建`custom.js`文件到`wwwroot/swagger-ui/`目录下，目录没有自己建，当然js文件名自己随便取；
3. 编辑`custom.js`文件内容：



```

console.log("Custom JS has been loaded and executed.");  
 
const originalFetch = window.fetch;  
  
window.fetch = function(...args) {  
    const [resource, config] = args;  
  
    return originalFetch.apply(this, args).then(response => {  
        // 检查请求的 URL        
        const requestUrl = new URL(response.url, window.location.origin).pathname;  
        if (requestUrl.endsWith("/login")) {  
            // 克隆响应以便读取  
            response.clone().json().then(data => {  
                const token = data.result.token;  
                if (token) {  
                    console.log("Token received via fetch override:", token);  
                    authorizeSwagger(token);  
                } else {  
                    console.warn("Token not found in login response.");  
                }            
            }).catch(err => {  
	            console.error("Failed to parse login response:", err);  
            });        
        }        
        return response;  
    });
}
  
// 定义授权函数  
function authorizeSwagger(token) {  
    const bearerToken = 'Bearer ' + token;  
    console.log("Setting Swagger UI Authorization with token:", bearerToken);  
    if (window.ui && window.ui.authActions) {  
        window.ui.authActions.authorize({  
            Bearer: {  
                name: 'Bearer',  
                schema: {  
                    type: 'apiKey',  
                    in: 'header',  
                    name: 'Authorization',  
                    description: '',  
	            },                
	            value: bearerToken,  
	        }        
        });        
        console.log("Authorization set successfully");  
    } else {  
        console.warn("Swagger UI authActions not available yet.");  
    }
}
    

```

##### 第二步：启用静态文件服务


在`Program.cs` 中，启用静态文件服务；



```
var app = builder.Build();

// 启用静态文件服务
app.UseStaticFiles();

```

##### 第三步：SwaggerUI 中间件中注入 js 脚本


使用`InjectJavascript`的方式将上面的`custom.js`注入到SwaggerUI中间件中。



```
// 启用Swagger  
app.UseSwagger();  
app.UseSwaggerUI(options =>  
{  
    options.InjectJavascript("/swagger-ui/custom.js");  
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1");  
});

```

OK!


大家快来试试吧\~


![](https://img2024.cnblogs.com/blog/665705/202412/665705-20241213155618007-774632121.png)


 本博客参考[FlowerCloud机场订阅官网](https://hanlianfangzhi.com)。转载请注明出处！
