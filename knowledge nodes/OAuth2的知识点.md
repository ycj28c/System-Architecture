OAuth2的应用
-------------
OAuth2主要应用于第3方授权的情况，解决的是一个用户对应用的信任问题。  
比如RubyChina需要登录才能使用，但是用户不信任RubyChina，因为登录需要保存用户名密码，但是使用OAuth2就可以通过Github的授权登录，RubyChina获得授权后可以访问部分用户在Github的资源（比如头像，用户名之类），并且可以登录RubyChina和访问RubyChina的资源。这样用户无需RubyChina的账号照样访问资源，既解决了用户的信任问题，同时能让用户更方便的登录从而增加了新用户数量。  

重要名词
------
这里是我的理解：  
（1）Client： 第三方应用，或者说客户端，这里就是RubyChina。  
（2）Resource Owner(user)： 资源所有者，就是用户User自己。   
（3）Authorization server： 认证服务器，就是Github的认证服务。  
（4）Resource server： 资源服务器，就是Github的昵称头像之类。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。  
（5）User Agent：用户代理，可以是浏览器或者是手机app。可以和Client归到一类去。  

基本流程
-------
1.用户user通过客户端client（RubyChina）向Github的Resource Owner（也就是Github中的用户帐号）请求授权  
2.Github的用户帐号同意授权，将授权信息（可能是code或者其他形式）发送给Client  
3.客户端Client（RubyChina）使用上一步获得的授权，向Github认证服务器申请token  
4.Github认证服务器对上一步的请求认证之后，同意并发送token给客户端Client（RubyChina）  
5.客户端获得token，可以使用这个token（有时间限制）向Resource Server（Github的资源服务器）申请帐号名头像等信息  
6.Githu资源服务器Resource Server开放并发送用户请求的资源  

授权模式
--------
上述基本流程中第2）步是有多种授权模式的：  
1.授权码模式（authorization code）  
2.简化模式（implicit）  
3.密码模式（resource owner password credentials）  
4.客户端模式（client credentials）  

其中授权码模式是最完整最推荐的流程。就是完整的上述基本流程内容，其中RubyChina客户端向Github用户帐号请求的时候，需要发送这样的HTTP信息:
* response_type：表示授权类型，必选项，此处的值固定为"code"  
* client_id：表示客户端的ID，必选项  
* redirect_uri：表示重定向URI，可选项  
* scope：表示申请的权限范围，可选项  
* state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。  

注意这里的clientId是由RubyChina生成的，为了获得这个ClientId，RubyChina开发者需要到Github注册Oauth application。需要填写Application name，Homepage URL，Application description，Authorization callback URL。通过申请以后Github就会颁发给RubyChina开发者ClientId（唯一标识了RubyChina）和ClientSecret（获取token进行加密）。

这里Github的用户同意授权后，Github会发送code给设置的Authorization callback URL去。然后Ruby China就可以使用code+clientId+clientSecret作为paramter向Github的认证服务器请求token了。然后这个token就可以在规定时间内随意使用权限范围内的用户信息了。

其他的模式就是精简版的了，比如简化模式就是省去了授权码这一步，在Github用户授权后，Github认证服务器直接返回token（这样会暴露token给访问者，token会出现在返回的重定向url中，本来是应该在后台完成）。而密码模式就是用户user向Client（Ruby China）提供Github的用户名密码，然后再请求Github（鸡肋的模式，因为暴露了用户名密码给客户端了，除非这个客户端是操作系统级别或者google级别的高可信客户端）。而客户端模式就类似于Client（Ruby China）的后台登陆了，和Oauth其实关系不大。

总结一下：  
1.授权码模式： 正宗的OAuth认证，推荐  
2.密码模式： 为遗留项目设计  
3.简化模式： 为Web浏览器设计  
4.客户端模式： 为后台API服务消费者设计  

此外这些模式中都使用了token，而token是有时间限制的，通常会设置在10分钟左右。为了良好的用户体验，用户登录期间可以通过更新token来继续访问。具体的做法就是在Github认证服务器发送回Client（Ruby China）的时候会包含refresh_token参数，这个参数可以用来获取下一次的token，所以只要用户活跃在Client（Ruby China），也就是用户仍在在Ruby China的后台session中，就可以调用refresh_token继续或者token和refresh_token。

Reference
---------
[理解OAuth 2.0](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)  
[OAuth2.0原理和验证流程分析](https://www.jianshu.com/p/d74ce6ca0c33)  
[彻底理解 OAuth2 协议](https://www.youtube.com/watch?v=T0h6A-M_WmI)  

