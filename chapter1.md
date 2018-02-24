# OIDC在 ASP.NET Core中的应用

我们在《ASP.NET Core项目实战的课程》第一章里面给identity server4做了一个全面的介绍和示例的练习 。

如果想完全理解本文所涉及到的话题，你需要了解的背景知识有：

* 什么是OpenId Connect \(OIDC\)

* OIDC 对oAuth进行了哪些扩展？

* Identity Server4又是什么

* ASP.NET Core的权限体系中的OIDC认证框架（客户端）

* Identity Server4提供的OIDC认证服务（服务端）

* 客户端与服务端的集成

### 什么是 OIDC

在了解OIDC之前，我们先看一个很常见的场景。假使我们现在有一个网站要集成微信或者新浪微博的登录，两者现在依然采用的是oAuth 2.0的协议来实现 。 关于微信和新浪微博的登录大家可以去看看它们的开发文档。

在我们的网站集成微博或者新浪微博的过程大致是分为五步：

1. 准备工作：在微信/新浪微博开发平台注册一个应用，得到AppId和AppSecret

2. 发起 oAauth2.0 中的 Authorization Code流程请求Code

3. 根据Code再请求AccessToken（通常在我们应用的后端完成，用户不可见）

4. 根据 AccessToken 访问微信/新浪微博的某一个API，来获取用户的信息

5. 后置工作：根据用户信息来判断是否之前登录过？如果没有则创建一个用户并将这个用户作为当前用户登录（我们自己应用的登录逻辑，比如生成jwt），如果有了则用之前的用户登录。

中间第2到3的步骤为标准的oAuth2 授权码模式的流程，如果不理解的可以参考阮一峰所写的《[理解oAuth2.0 ](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)》一文。我们主要来看第4和5步，对于第三方应用要集成微博登录这个场景来说最重要的是我希望能快速拿到用户的一些基本信息（免去用户再次输入的麻烦）然后根据这些信息来生成一个我自己的用户跟微博的用户Id绑定（为的是下次你使用微博登录的时候我还能把你再找出来）。

oAuth在这里麻烦的地方是我还需要再请求一次API去获取用户数据，注意这个API和登录流程是不相干的，其实是属于微博开放平台丛多API中的一个，包括微信开放平台也是这样来实现。这里有个问题是前面的 2和3是oAuth2的标准化流程，而第4步却不是，但是大家都这么干（它是一个大家都默许的标准）

于是大家干脆就建立了一套标准协议并进行了一些优化，它叫OIDC

> OIDC 建立在oAuth2.0协议之上，允许客户端\(Clients\)通过一个授权服务\(Authorization Server\)来完成对用户认证的过程，并且可以得到用户的一些基本信息包含在JWT中。

# OIDC对oAuth进行了哪些扩展？

在oAuth2.0授权码模式的帮助下，我们拿到了用户信息。

![](/assets/authroization_code_flow)

以上没有认证的过程，只是给我们的应用授权访问一个API的权限，我们通过这个API去获取当前用户的信息。而OIDC给oAuth2进行扩展之后就填补了这个空白。它添加了以下两个内容：

* response\_type 添加IdToken
* 添加userinfo endpoint，用idToken可以获取用户信息

我们先来看看IdToken

在oAuth2的授权码模式的第一步，我们向authorize endpoint请求code的时候所传递的response\_type表示授权类型，原来只有固定值code。

```
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
```

OIDC对它进行了扩展，现在你有三个选择：code, id\_token和 token，现在我们可以这样组合来使用。

| "response\_type" value | Flow |
| :--- | :--- |
| code | Authorization Code Flow |
| id\_token | Implicit Flow |
| id\_token token | Implicit Flow |
| code id\_token | Hybrid Flow |
| code token | Hybrid Flow |
| code id\_token token | Hybrid Flow |

Implicit Flow也支持返回id token 同时新增 Hybird Flow

| Property | Authorization Code Flow | Implicit Flow | Hybrid Flow |
| :--- | :--- | :--- | :--- |
| access token和id token都通过Authorization endpoint返回 | no | yes | no |
| 两个token都通过token end point 返回 | yes | no | no |
| 用户使用的端\(浏览器或者手机）无法查看token | yes | no | no |
| Client can be authenticated | yes | no | yes |
| 支持刷新token | yes | no | yes |
| 不需要后端参与 | no | yes | no |
|  |  |  |  |

  


  


























































































































