# Nginx Error Page配置


### 语法
```
error_page code [ code... ] [ = | =answer-code ] uri |@named_location 
```

### 示例
1. 作用是当发生错误的时候能够显示一个预定义的uri
```
error_page 502 503 /50x.html;
location = /50x.html {
    root /usr/share/nginx/html;
}  
```
这样实际上产生了一个内部跳转(internal redirect)，当访问出现502、503的时候就能返回50x.html中的内容，这里需要注意是否可以找到50x.html页面，所以加了个location保证找到你自定义的50x页面。
 
2. 同时我们也可以自己定义这种情况下的返回状态吗，比如：
```
error_page 502 503 =200 /50x.html;
location = /50x.html {
    root /usr/share/nginx/html;
} 
```
这样用户访问产生502 、503的时候给用户的返回状态是200，内容是50x.html。

3. 当error_page后面跟的不是一个静态的内容的话，比如是由proxyed server或者FastCGI/uwsgi/SCGI server处理的话，server返回的状态(200, 
302, 401 或者 404）也能返回给用户。
```
error_page 404 = /404.php;
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}  
```
4. 也可以设置一个named location，然后在里边做对应的处理。
```
error_page 500 502 503 504 @jump_to_error;
location @jump_to_error {    
    proxy_pass http://backend;
}
```
5. 同时也能够通过使客户端进行302、301等重定向的方式处理错误页面，默认状态码为302。
```
error_page 403      http://example.com/forbidden.html;
error_page 404 =301 http://example.com/notfound.html;
```
同时error_page在一次请求中只能响应一次，对应的nginx有另外一个配置可以控制这个选项：recursive_error_pages
默认为false，作用是控制error_page能否在一次请求中触发多次。

### Nginx 自定义404错误页面配置中有无等号的区别
error_page 404 /404.html 可显示自定义404页面内容，正常返回404状态码。

error_page 404 = /404.html 可显示自定义404页面内容，但返回200状态码。

error_page 404 /404.php 如果是动态404错误页面，包含 header 代码（例如301跳转），将无法正常执行。正常返回404代码。

error_page 404 = /404.php 如果是动态404错误页面，包含 header 代码（例如301跳转），加等号配置可以正常执行，返回php中定义的状态码。但如果php中定义返回404状态码，404状态码可以正常返回，但无法显示自定义页面内容（出现系统默认404页面），这种情况可以考虑用410代码替代（ header("HTTP/1.1 410 Gone"); 正常返回410状态码，且可正常显示自定义内容）。

{{<admonition >}}
由于在nginx配置中，设置了limit_req的流量限制，导致许多请求返回503错误代码，在限流的条件下，为提高用户体验，希望返回正常Code 200，且返回操作频繁的信息：
{{</admonition >}}
```
location  /test {
  ... 
  limit_req zone=zone_ip_rm burst=1 nodelay; 
  error_page 503 =200 /dealwith_503?callback=$arg_callback;
}
location /dealwith_503{ 
  set $ret_body '{"code": "V00006","msg": "操作太频繁了，请坐下来喝杯茶。"}';
   if ( $arg_callback != "" ) 
   { 
       return 200 'try{$arg_callback($ret_body)}catch(e){}'; 
   } 
   return 200 $ret_body; 
}
```
