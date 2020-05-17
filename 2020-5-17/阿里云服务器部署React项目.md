前言：花费了三天的时间在@Eric的指导下不断的踩坑，总算把项目配置完毕了，特此记录，分享此刻舒坦的心情!



## **一、阿里云实例的配置**

首先注册登录阿里云，购置云服务器，学生则可以拉到最下方的【更多推荐】里有一项【学生机】![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/C5I6tuQmWty2LHkfPkGvPrHhII2HVymhs7tPg60hcjQfG58LtbM24AhJ5EEIgMksFdWUBpB5l4XhQyU3MoU*Lw!!/mnull&bo=7wQ*AQAAAAADB*c!&rf=photolist&t=5) 

领券购买114元一年的云服务器ECS，领20元券，勾选镜像，我们选的是ubuntu18.0.4 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/C5I6tuQmWty2LHkfPkGvPgF3QIegli*YOjS6VBuPqfdaM7IijmKClDug3fBtAKvm4Qb3x5wfcu*xznUMz0XczA!!/b&bo=9QTyAQAAAAADByA!&rf=viewer_4) 

购买完了之后打开阿里云控制台，点击云服务器ECS找到刚才购买的实例，一开始会让你设置用户密码，不能忘记，后面远程登录需要使用。

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4Nc9Mdu1ihePEH7GoRxc1ybK3oMhvyiHzBBWUTtZiDD2Mp2jbo3zCInTQcQ5jC6pcKY8SLp28rkpP5XnE3NzHCn4!/b&bo=QAbbAgAAAAADF60!&rf=viewer_4) 

## **配置密钥对**

点击最左边伸缩导航栏的【密钥对】选项

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NbYsMFaCFOIMJa1x0vWgCP5xOoCHI5.kWJQFJNNq*7QwRsyklZHfPA*uq8IIOJWwh5Nc3k331T14BhUoM5OgSiM!/b&bo=4AHbAgAAAAADFwo!&rf=viewer_4) 

然后点击右上角的创建密钥对，勾选【导入已有密钥对】,打开C:/用户/user/.ssh文件夹，看看有没有id_rsa.pub这个文件如果没有需要手动生成,有的话直接跳到后一步。
没有id_rsa.pub的话打开控制台，在控制台中输入以下命令:

``` cmd
$ ssh-keygen -t rsa -C "youremail@example.com"
```

密钥类型可以用 -t 选项指定。如果没有指定则默认生成用于SSH-2的RSA密钥。这里使用的是rsa密钥。
同时在密钥中有一个注释字段，用-C来指定所指定的注释，可以方便用户标识这个密钥，指出密钥的用途或其他有用的信息。所以在这里输入自己的邮箱或者其他都行,当然，如果不想要这些可以直接输入：

```cmd
$ ssh-keygen
```

自己设置一个密钥对名称，将id_rsa.pub中的所有内容（本机的RSA公钥）复制到下方的公钥内容里面，其他不用管，点击确定，密钥对就创建好了。

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NQN.7jNBcy3y2*Qn95rFL5muopl0AzW6yotqaZWAqqAN7qjMU*DD3nMg4EFAv3a8Yq28CYUZaK2yxpLiqbYrYPs!/b&bo=tALGAQAAAAADF0M!&rf=viewer_4) 


## **绑定密钥对**
![image-20200515112109883](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/C5I6tuQmWty2LHkfPkGvPmNH7uWQjZD5RcrmoCIP9z0qF6Bs7gXF4GRDE5Wn*Biu01hPZIUvSar1IrG8ukjxTQ!!/b&bo=VQUvAQAAAAADB1w!&rf=viewer_4)



但是注意阿里云上绑定的密钥对只能是一对一的,即一个实例只能绑定一个密钥对,如果还希望别的终端可以ssh登录服务器,则需要在服务器的/root/.ssh文件夹下配置authorized_keys文件,将需要连接的主机SSH公钥复制到里面(放到最下面一行)

然后我们就可以进行远程连接了，我们选择vscode的控制台进行远程连接，在控制台输入以下命令：

```cmd
ssh 实例用户名(默认为root)@实例公网ip（如39.97.175.30）   
```

然后输入实例密码就可以登陆了，出现如下信息则登录成功。![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NULRSN*i5SWD7AAIj8NXIDTgQPdegjZBZGkiLSR*smgAK5mNjeC4CEqYdK7ZjhbYPX.BY2FKCrblxI3GAUDVFHY!/b&bo=qQIJAQAAAAADF5E!&rf=viewer_4)

然后vscode安装remote-SSH插件后则可以进行可视化操作。

 

 最后，在阿里云控制台下找到找到【网络和安全】点击下面的【安全】菜单【创建安全组】创建之后记得【配置规则】如下图。![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NUUOJfGbvjGfi6xm7xHZC3BzDakOd3TrEITpVPFMinIpPQw705HQhtPhihR8QiQQLgI6cMQnYxg6UDIYn*JQXeg!/b&bo=RAWaAQAAAAADF.g!&rf=viewer_4)

 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4Nc4ZqVs0K4tcx9d5JTbZJ6LjHp4PgV7hA7b0XZihP0OFUgJg41sP59nFZ5a8FIoUlG4M.ojyR.sJ9d0JB1hkovQ!/b&bo=pALZAQAAAAADF0w!&rf=viewer_4) 

## **配置安全组规则**

# ![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NX95aMjGDNRbu6Th0n4GFecq1wWaAG9AvmZl4VgbReb8dUyNXAl2FYtlKvv2pp0snpHtr0WYsoL2ce3MeKiK05w!/b&bo=dgQbAQAAAAADF1o!&rf=viewer_4)

## **配置规则可参考阿里云的帮助文档**

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NTWut1eK9fQJrEvfbpXvuDaeKXtYVszAAjmM2lSGCowIXckXoc4zhvxs*R4EHl5EemWZxD4*gqSgYL1ATTGi0nw!/b&bo=qgSqAQAAAAADJwc!&rf=viewer_4) 

配置完端口后，阿里云服务器的实例配置就算完成了

 

## **二、apt-get安装mysql、npm、git、pm2、nginx**

## **配置mysql**

首先是安装mysql，我们可以使用百度经验上的[教程](https://jingyan.baidu.com/article/c45ad29c71baf4051753e2b8.html)

安装完了以后尝试使用本机远程访问服务器的mysql数据库

 

## 修改监听端口（不设置可能会连接失败）

  修改MySQL配置文件/etc/my.cnf，在[mysqld]找到如下配置:

```cmd
 bind-address = 127.0.0.1
```

  注释掉该行或改成0.0.0.0，重启mysql生效。

 

## 给远程用户授权:

  **# %表示所有客户端IP**

```cmd
  mysql> grant all on *.* to user_name@"%" identified by "user_password"; 
```

 

  **# 刷新权限：**

```cmd
  FLUSH PRIVILEGES;
```

 重启mysql服务



## 在本地远程连接服务器mysql

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NQfIrj4Sudhfjb*NJMv.I3WvKjd3pFsIb52xBZTkOdbgPJcpnRkjEWRkbaxYxUm85jotGxrDzuLj6jCdofN*1pc!/b&bo=0AKPAQAAAAADF24!&rf=viewer_4) 



## apt-get安装npm、git、pm2、nginx

安装过程可以搜索教程，这里略过。

推荐教程：https://my.oschina.net/litengteng/blog/900078

https://my.oschina.net/litengteng/blog/900078

 

## **pm2自动部署**

这里搬运了大佬@Eric的项目ecosystem.config.js文件，@Eric也算是我的js开发的启蒙老师了。

```javascript
module.exports = {

 apps: [

  {

   name: 'network-bureau-review-com', // 项目名称

 

   // Options reference: https://pm2.keymetrics.io/docs/usage/application-declaration/

   args: 'one two',

   instances: 1,

   autorestart: true,

   watch: false,

   max_memory_restart: '1G',

  }

 ],

 

 deploy: {

  production: {

   user: 'root', // 实例用户名

   host: '39.97.175.30', // 实例公网ip

   ref: 'origin/master', // 选择项目需要配置的git分支

   repo: 'git@github.com:Zbr1920410104/network-bureau-review-com.git', // 项目仓库地址

   path: '/network-bureau/network-bureau-com-review', // 服务器项目创建目录，没有目录会自己新建

   'post-deploy':

     'npm install && npm run build', // 配置过程

   'post-setup': 'npm install && npm run build' // 升级过程

  }

 }

};

```

 全部配置完了以后本地package.json文件里面设置部署命令

```javascript
"scripts": {

  "initDeploy": "pm2 deploy production setup",

  "deploy": "pm2 deploy production update"

 },
```

项目前端在本地运行npm run initDeploy，找前端文件夹，看看有没有current/build/index.html这个文件，有的话项目打包成功。

后端运行npm run initDeploy之后还需要添加key.js数据库密码文件，然后初始化数据库，这里就算配置完成了。

 

## **配置nginx**

这地方真的全靠@Eric帮我，捋清了逻辑，配置好

@Eric提供的[参考资料](https://juejin.im/post/5ea931866fb9a043815146fb)

 

找到nginx配置文件etc/nginx/nginx.conf，然后修改配置（在网页配置中加入server配置），因为我们是单网页项目，我们添加81端口，这样81端口监听的就是我们部署好的项目了。

```cmd
server {
    listen    81; # 端口
    server_name 39.97.175.30; # 主机ip   

    # charset koi8-r;

    # access_log logs/host.access.log main;

    location / {

      root /network-bureau/network-bureau-com-review/current/build;       # 项目路径
      index index.html;            
      try_files $uri $uri/ /index.html @rewrites; 
      
      expires -1;             # 首页一般没有强制缓存
      add_header Cache-Control no-cache;
    }


    location @rewrites {
      rewrite ^(.+)$ /index.html break;
    } 
    
} 
```

 

## **三、后续的项目升级**

项目修改完本地git push了之后会，无需再使用npm run initDeploy指令，直接在本地运行npm run deploy即可在线升级。

部署完之后项目如果有报错可以在/root/.pm2/logs里面的日志文件里查询。



参考：https://blog.csdn.net/qq_30865575/article/details/78273291



​                                                                                                                                           14/5/2020 Brown At Huaian