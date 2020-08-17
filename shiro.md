### shiro

####shiro简介

Apache Shiro 是 Java 的一个安全框架。shiro可以帮助我们完成：认证，授权，加密，会话管理，与Web集成，缓存等。

####shiro工作原理

![2](C:\Users\zhangruijsqt\Desktop\day\pic\2.png)





Subject:主体，就是当前用户，所有与shiro交互的程序都是subject。

SecurityManager：安全管理器，将subject发送的验证和授权信息与后面的Realm读取到的信息进行验证。

Realm: 域，相当于DateSource，SecurityManager从Realm获得用户信息来完成Subject的验证和授权。

#### shiro身份验证

shiro身份验证通过用户提供的principals(身份)和credentials(证明)来验证用户的身份。

![身份验证](C:\Users\zhangruijsqt\Desktop\day\pic\\身份验证.png)

身份验证步骤：

1. 首先Subject.log(token)进行登录，subject会委托给SecurityManager。
2. SecurityManager负责真正的验证逻辑，它会委托给Authenticator进行验证。
3. Authenticator是Shiro核心身份验证者，可以自定义是实现该接口。
4. Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认 ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证。
5. Authenticator 将token传入Realm，从Realm中获取身份验证信息进行验证。此处可以配置多个Realm，按照一定的策略来进行访问。

#### shiro授权

shiro授权就是权限访问控制，通过它来判断用户可以访问哪些资源。需要了解的基本概念：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。

Subject：主体，就是用户。

Resource：资源，用户可以访问和操作的所有东西，包括页面，编辑数据。只有授权后才可以访问。

Permission：权限，表示在应用中用户能不能访问某个资源 。通常权限包括：访问页面，CRUD等等。

Role：角色，角色代表了权限的集合，一般来说我们会赋予用户角色而不是权限，这样会比较方便。

**授权流程**



![授权流程](C:\Users\zhangruijsqt\Desktop\day\pic\\授权流程.png)



1. 首先调用subjuct.isPermitted* /hasRole*接口。
2. Subject委托给SecurityManager，SecurityManager委托给Authorizer。
3. Authorizer会把字符串转换成对应的实例，比如调用isPermitted("user:view"),会通过PermissionResolver转换成permission实例。
4. 授权之前调用相应的Realm进行身份验证。Authorizer会判断Realm的角色/权限是否匹配。如果有多个Realm，会委托给ModularRealmAuthorizer来进行判断。

> 自定义Realm时，选择继承 AuthorizingRealm 而不是实现 Realm 接口。因为：  AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)：表示获取身份验证信息；AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals)：表示根据用户身份获取授权信息。这种方式的好处是当只需要身份验证时只需要获取身份验证信息而不需要获取授权信息 

#### shiro ini配置

1. [main]部分：提供了根对象securityManager及其依赖对象的配置。

2. [users]部分：配置用户名/密码及其角色。用户名=密码，(角色1,角色2......)；如

   ```java
   [users]
   zhang=123,role1,role2
   wang=123;
   ```

3. [roles]部分：配置角色和权限的关系。角色=权限1，权限2......；如

   ```java
   role1=user:create,user:update
   role2=*;
   ```

4. [urls]部分：配置url和对应的拦截器之气那的关系。

