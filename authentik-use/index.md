# Authentik使用


## Proxy Provider
{{< figure src="/images/authentik_proxy.png" title="Proxy Provider (process)" >}}
代理前哨设置以下用户特定的标头：

`X-authentik-username`当前登录用户的用户名

`X-authentik-groups`用户所属的组，用竖线分割

`X-authentik-email`当前登录用户的邮箱地址

`X-authentik-name`当前登录用户的全名

`X-authentik-uid`当前登录用户的哈希标识符

应用程序特定的标头

`X-authentik-meta-outpost`前哨名称

`X-authentik-meta-provider`提供者名称

`X-authentik-meta-app`slug

`X-authentik-meta-version`版本

`X-Forwarded-Host`客户端发送原始 Host标头

附加：

此外，您可以设置additionalHeaders组或用户的属性来设置其他标头：
```
additionalHeaders:
    X-test-header: test-value
```
### 退出登录 ####
当您在没有有效 cookie 的情况下访问域时，会自动完成登录.

使用单应用程序模式时，导航至app.domain.tld/outpost.goauthentik.io/sign_out.

使用域级模式时，导航到auth.domain.tld/outpost.goauthentik.io/sign_out，其中 auth.domain.tld 是为提供商配置的外部主机。

要注销，请导航至/outpost.goauthentik.io/sign_out。

### 转发认证 ####
***Nginx*** 
```
server {
    listen                  443 ssl http2;
    server_name             _;

    ssl_certificate         /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key     /etc/ssl/private/ssl-cert-snakeoil.key;

    proxy_buffers 8 16k;
    proxy_buffer_size 32k;

    location / {
        # proxy_pass http://localhost:5000;
        # proxy_set_header Host $host;
        # proxy_set_header ...

        ##############################
        # authentik-specific config
        ##############################
        auth_request     /outpost.goauthentik.io/auth/nginx;
        error_page       401 = @goauthentik_proxy_signin;
        auth_request_set $auth_cookie $upstream_http_set_cookie;
        add_header       Set-Cookie $auth_cookie;

        # translate headers from the outposts back to the actual upstream
        auth_request_set $authentik_username $upstream_http_x_authentik_username;
        auth_request_set $authentik_groups $upstream_http_x_authentik_groups;
        auth_request_set $authentik_email $upstream_http_x_authentik_email;
        auth_request_set $authentik_name $upstream_http_x_authentik_name;
        auth_request_set $authentik_uid $upstream_http_x_authentik_uid;

        proxy_set_header X-authentik-username $authentik_username;
        proxy_set_header X-authentik-groups $authentik_groups;
        proxy_set_header X-authentik-email $authentik_email;
        proxy_set_header X-authentik-name $authentik_name;
        proxy_set_header X-authentik-uid $authentik_uid;
    }

    location /outpost.goauthentik.io {
        proxy_pass              http://outpost.company:9000/outpost.goauthentik.io;
        # 确保此虚拟服务器的主机与您配置的外部 URL 匹配
        proxy_set_header        Host $host;
        proxy_set_header        X-Original-URL $scheme://$http_host$request_uri;
        add_header              Set-Cookie $auth_cookie;
        auth_request_set        $auth_cookie $upstream_http_set_cookie;
        proxy_pass_request_body off;
        proxy_set_header        Content-Length "";
    }

    location @goauthentik_proxy_signin {
        internal;
        add_header Set-Cookie $auth_cookie;
        return 302 /outpost.goauthentik.io/start?rd=$request_uri;
    }
}
```
***Nginx代理服务器***
```
proxy_buffers 8 16k;
proxy_buffer_size 32k;

port_in_redirect off;

location / {
    # Put your proxy_pass to your application here
    proxy_pass          $forward_scheme://$server:$port;
    # Set any other headers your application might need
    # proxy_set_header Host $host;
    # proxy_set_header ...

    ##############################
    # authentik-specific config
    ##############################
    auth_request     /outpost.goauthentik.io/auth/nginx;
    error_page       401 = @goauthentik_proxy_signin;
    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header       Set-Cookie $auth_cookie;

    # translate headers from the outposts back to the actual upstream
    auth_request_set $authentik_username $upstream_http_x_authentik_username;
    auth_request_set $authentik_groups $upstream_http_x_authentik_groups;
    auth_request_set $authentik_email $upstream_http_x_authentik_email;
    auth_request_set $authentik_name $upstream_http_x_authentik_name;
    auth_request_set $authentik_uid $upstream_http_x_authentik_uid;

    proxy_set_header X-authentik-username $authentik_username;
    proxy_set_header X-authentik-groups $authentik_groups;
    proxy_set_header X-authentik-email $authentik_email;
    proxy_set_header X-authentik-name $authentik_name;
    proxy_set_header X-authentik-uid $authentik_uid;
}

location /outpost.goauthentik.io {
    proxy_pass              http://outpost.company:9000/outpost.goauthentik.io;
    # 确保此虚拟服务器的主机与您配置的外部 URL 匹配
    proxy_set_header        Host $host;
    proxy_set_header        X-Original-URL $scheme://$http_host$request_uri;
    add_header              Set-Cookie $auth_cookie;
    auth_request_set        $auth_cookie $upstream_http_set_cookie;
    proxy_pass_request_body off;
    proxy_set_header        Content-Length "";
}

location @goauthentik_proxy_signin {
    internal;
    add_header Set-Cookie $auth_cookie;
    return 302 /outpost.goauthentik.io/start?rd=$request_uri;
}
```

## Oauth2 Provider
### 前期准备

1. 配置 authentik的oauth2 provider
{{< figure src="/images/oauth_provider.png" title="Provider Setting Page" >}}
重定向URI填写的是 portainer的主页地址
2. 配置 application
{{< figure src="/images/oauth_application.png" title="Application Setting Page" >}}
选择新增好的provider

### 配置portainer
我们使用 portainer作为演示
1. 安装 portainer

这里我们使用 docker进行安装
```
docker volume create portainer_data
docker run -d -p 9000:9000 --name=portainer --restart=unless-stopped 
-v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```
2. 登录portainer平台、

在浏览器中访问 http://localhost:9000 来登录 Portainer，首次使用，需要为 admin用户设置密码。

3. 点击设置里的认证模块
{{< figure src="/images/portainer_oauth2.png" title="Portainer Setting Page" >}}
4. 配置对应的配置项
{{< image src="/images/portainer_oauth2_setting.png" height= "500" width= "1000" title="Portainer Setting Page" >}}
5. 登出 portainer，出现 Login With Oauth
{{< figure src="/images/portainer_login.png" title="Portainer Login Page" >}}
6. 点击Login With Oauth，会跳转到 authentik页面进行授权，授权通过后就能登录到 portainer首页
{{< figure src="/images/oauth_login.png" title="Authentik Login Page" >}}
{{< figure src="/images/portainer_login_success.png" title="Portainer Login Success" >}}


## LDAP Provider
### 前期准备
1. 新增 ldap flow
2. 配置 authentik的 ldap provider、 application

[配置 authentik](https://docs.goauthentik.io/docs/providers/ldap/generic_setup)

### 测试
#### ldapsearch
```
ldapsearch -x -H ldap://ladp_addr:389 -D 'cn=akadmin,ou=users,dc=ldap,dc=goauthentik,dc=io' 
-b 'ou=users,dc=ldap,dc=goauthentik,dc=io' -w 'xxx@123' "cn=akadmin"
```

#### jumpserver 
我们使用 jumpserver作为演示ldap登录
{{<admonition >}}
安装 jumpserver,详见[docker安装 jumpserver](https://github.com/jumpserver/Dockerfile)
{{</admonition >}}

{{<figure src="/images/ldap_jumpserver_setting.png" title="Jumpserver ldap setting Page">}}

配置完之后就能用 authentik用户登录 jumpserver了

