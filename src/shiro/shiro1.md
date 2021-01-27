---
title: "Shiro(一)"
date: 2020-12-07T16:07:07+08:00
tags: ["shiro", "security"]
categories: ["java"]
draft: false
---

#  shiro(一)

>最近项目里用到了shiro这个安全框架，之前没接触过安全框架，所以对一些基本概念的理解很是费劲，网上也搜了不少教程，最后实现起来也是磕磕绊绊，所以借此机会好好总结一下：**什么是shiro**、**springboot整合shiro如何实现**，另外有时间的话，再针对**核心源码**进行解读。



----

## shiro是什么

#### 官方回答如下：

>**Apache Shiro™** is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.

>Apache Shiro 是一个功能强大且易于使用的Java安全框架，它可以用于身份验证、授权、加密和会话管理。基于Shiro易于理解的API，您可以快速、轻松地保护任何应用程序——从小型的移动端程序到大型的web和企业应用程序。



#### **几个核心概念**

* **Subject**

  Subject这个词是安全专业的专业术语，它指代**当前正在运行（访问）的对象**，这个对象可以是用户、第三方进程、守护进程账户或者类型的一些东西。Java程序员可以从Session角度理解。当一个用户成功登录的时候，传统的会话记录方式就是往session里存上`<K, V>`，这里的V可以理解成是Subject，这个V可以标识当前用户。

  ```
  import org.apache.shiro.subject.Subject;
  import org.apache.shiro.SecurityUtils;
  ...
  Subject currentUser = SecurityUtils.getSubject();
  ```

* **SecurityManager**

  SecurityManager对Subject进行统一配置、管理。是Shiro里最核心的一个组件。

* **Realms**

  Realm中文意思就是“**域**”。它是Shiro与应用程序间的“桥”或“连接器”。需要进行身份验证和授权时候，Shiro会从一个或多个域中查找所需内容。从某种意义上来说，它就是一个特定的DAO层，用以调用底层数据库。

  Shiro还提供了一些开箱即用的数据源，如LDAP、JDBC、INI等

  >```
  >[main]
  >ldapRealm = org.apache.shiro.realm.ldap.JndiLdapRealm
  >ldapRealm.userDnTemplate = uid={0},ou=users,dc=mycompany,dc=com
  >ldapRealm.contextFactory.url = ldap://ldapHost:389
  >ldapRealm.contextFactory.authenticationMechanism = DIGEST-MD5 
  >```

  **TIps:**在网上你或许也会看到很多shiro教程是这种格式的。因为Shiro的很多组件时可以通过配置文件来实现的，在某些应用领域是很方便的。当然对于注解爱好者的开发人员来说，它也是支持大量的注解，在springboot整合shiro章节再进行详述。



#### **Authentication**

**身份验证**。也就是平常写的登录接口的职能——验证用户名、密码。

在代码中就是要**实现相应接口，然后实现身份验证的方法**。

当然也可以将身份验证的步骤进行细分，分成用户名验证、密码匹配验证等形式，这样做的目的可能是为了遵守”单一原则“的设计模式吧。

例如我的项目中，因为密码是加密存在数据库的，而这个加密规则也不是一两行代码搞定的，所以在身份验证方法中只是验证用户名的有效性，然后再另一个方法里去验证密码的正确性。

```
Subject Login

//1. Acquire submitted principals and credentials:
AuthenticationToken token = new UsernamePasswordToken(username, password);
//2. Get the current Subject:
Subject currentUser = SecurityUtils.getSubject();
//3. Login:
currentUser.login(token);
```

往常的登录接口就可以简单写成这样，然后具体的校验交给自己实现的方法来完成。



#### Authorization

**授权**。这个单词和身份验证的单词很像。一般的系统都是**用户-角色-菜单（url）**的权限模式，一个用户会属于一个角色类型（Administrator除外），一种角色能访问的菜单也是配置好的。当用户登录成功后，会根据用户角色去查询用户的权限菜单，然后前端予以展示。

这样简陋的权限模块，对于稍微能抓点包，看得懂HTTP报文的人来说，在他用**领导账户**帮忙处理一点任务的时候，只要记住了请求报文重要信息，便可以**绕过前端页面的权限控制**。

所以呢，一般人会写个过滤器，每次请求都进行权限过滤。此法可行，但是不够灵活。比方说，我想针对每个接口进行针对性的过滤，或者某个接口只要拥有三个权限之一的权限就能访问，等等诸如此类的灵活管理就不太方便了。

高明一点的人，用自定义注解来实现。没错，shiro也提供了注解方式的权限配置。

```
    @RequiresPermissions(value = 
                             {"CLIENTINFO/FILE:SELECT","FILEINFO/SEARCH"}, 
                             logical = Logical.OR)
	@GetMapping("select")
	@ApiOperation("文件查看")
    public BaseResult select(@RequestParam String path,
                                   HttpServletResponse response) throws IOException {}
```

`@RequiresPermissions(value = {"CLIENTINFO/FILE:SELECT","FILEINFO/SEARCH"}, `

`logical = Logical.OR)`指明，用户要访问这个接口时候，只要具备其中一种权限即可。

接着，程序会去调用Realm中自己实现的授权方法

```
 	@Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        User user = (User)principalCollection.getPrimaryPrincipal();
        if (user == null) {
            log.error("shiro getAuthorizationInfo failure");
            return new SimpleAuthorizationInfo();
        }
        List<Resources> resources;
        if (Constants.USER_ISSUPPER == user.getIssupper()) {
            resources = resourcesService.selectAllResources();
        } else {
            resources = resourcesService.selectResourcesByUser(user.getUsername());
        }
        Set<String> permissions = 		
            resources.stream().map(Resources::getId).collect(Collectors.toSet());
        authorizationInfo.setStringPermissions(permissions);
        return authorizationInfo;
    }
```

`authorizationInfo.setStringPermissions(permissions);`将用户所拥有的权限Set赋值给授权信息对象。出于性能考虑，这里可以对资源集合获取增加Redis缓存。



#### Session Management

shiro的session管理是**独立于容器**的（Tomcat、Jetty等），方便跨端共享session。同时也可以自行实现session的缓存，这里缓存可以用Redis等NoSql数据库来进行存储。如果是集群、分布式环境需实现session的缓存管理相关方法。

```
Session session = subject.getSession();
session.getAttribute(“key”, someValue);
Date start = session.getStartTimestamp();
Date timestamp = session.getLastAccessTime();
session.setTimeout(millis);
```



#### Cryptography

加密。不用shiro的加密工具，就得自行使用JDK的MessageDigest类的静态方法来进行加密，相对来说比较麻烦

```
try {
    MessageDigest md = MessageDigest.getInstance("MD5");
    md.digest(bytes);
    byte[] hashed = md.digest();
} catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
} 
```

shiro以对象的形式来处理字符加密

```
String hex = new Md5Hash(myFile).toHex(); 
String encodedPassword =
    new Sha512Hash(password, salt, count).toBase64();
```

以上是shiro的一些常用的基本概念，如有误词，欢迎联系我指出。

