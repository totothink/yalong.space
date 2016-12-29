---
layout: post
title: "微信开放平台避免APP_Secret泄漏"
tags: []
---

1 微信开放平台OAuth实现方式

  微信OAuth2.0授权登录目前支持authorization_code模式，适用于拥有server端的应用授权。

获取access_token时序图：

  ![获取access_token时序图](/images/articles/wechat-oauth2-flow.png)


2 移动应用下发Secret的风险

  目前有些盆友在开发移动应用时，直接将APP_ID和APP_SECRET打包在APP里，这很容易导致APP_SECRET泄漏。
  泄露通常会带来资源滥⽤的后果，常见的有：
  （1）应⽤冒充：⽐如替换client_id和client_secret，将android⼿机应⽤装成iphone客户端，官⽅还不能封锁，因为这个client_id代表官⽅应⽤，封了等于断⾃⼰⽣意。
  （2）⾮合作数据挖掘：⽐如爬数据，又⽐如⾃动批量私信骚扰等。
  （3）⾮正常的⾼级接⼜调⽤。

3 如何实现APP云端转接（omniauth-oauth2）

  下发APP_SECRET的原因是因为微信用户授权后获得的是临时票据(code)，客户端需要拿APP_ID、APP_SECRET和Authorization code换取Access Token。
  为了避免APP_SECRET泄漏，微信建议将Appsecret、用户数据（如access_token）放在App云端服务器，由云端中转接口调用请求。

  以Ruby的omniauth-oauth2为例，服务端的callback接口接受Authorization Code，并向平台换取Access Token。

```ruby
    def build_access_token
      verifier = request.params["code"]
      client.auth_code.get_token(verifier, {:redirect_uri => callback_url}.merge(token_params.to_hash(:symbolize_keys => true)), deep_symbolize(options.auth_token_params))
    end
```
  这就意味着客户端在获得Authorization Code后，可以直接请求服务端omniauth指定的callback接口，服务端通过Authorization Code向平台换取Access Token。


参考资料
* [Oauth2.0安全案例回顾](http://php.ph/wydrops/drops/OAuth%202.pdf)
* [移动应用微信登录开发指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317851&token=&lang=zh_CN)
* [Omniauth](https://github.com/omniauth/omniauth) / [Omniauth-oauth2](https://github.com/intridea/omniauth-oauth2)
