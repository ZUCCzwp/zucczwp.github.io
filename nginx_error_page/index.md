# Nginx配置


### location 语法
#### 规则
```
location [=|~|~*|^~] /uri/ {
        ····· 
}
```
=     开头表示精确匹配

^~    开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）

~     开头表示区分大小写的正则匹配

~*    开头表示不区分大小写的正则匹配

!~    区分大小写不匹配的正则

!~*   不区分大小写不匹配的正则

/     通用匹配，任何请求都会匹配到

#### 匹配顺序：

首先匹配 "="，其次匹配 "^~", 其次是按文件中顺序的正则匹配，最后是交给 "/" 通用匹配。

当有匹配成功时候，停止匹配，按当前匹配规则处理请求

### rewrite 语法
规则：

last       – 基本上都用这个 Flag

break      – 中止 Rewirte，不在继续匹配

redirect   – 返回临时重定向的HTTP状态302

permanent  – 返回永久重定向的HTTP状态301

#### 可用来判断的表达式
-f 和 !-f    用来判断是否存在文件

-d 和 !-d    用来判断是否存在目录

-e 和 !-e    用来判断是否存在文件或目录

-x 和 !-x    用来判断文件是否可执行
#### 可用来判断的全局变量
```
例：http://localhost:88/test1/test2/test.php
$host：localhost
$server_port：88
$request_uri：http://localhost:88/test1/test2/test.php
$document_uri：/test1/test2/test.php
$document_root：D:\nginx/html
$request_filename：D:\nginx/html/test1/test2/test.php
```

### redirect 语法
```
server {
    listen 80;
    server_name start.igrow.cn;
    index index.html index.php;
    root html;
    if ($http_host !~ “^star\.igrow\.cn$&quot {
        rewrite ^(.*) http://star.igrow.cn$1 redirect;
    }
}
```

### 防盗链语法
```
location ~* \.(gif|jpg|swf)$ {
    valid_referers none blocked start.igrow.cn sta.igrow.cn;
    if ($invalid_referer) {
        rewrite ^/ http://$host/logo.png;
    }
}
```
### 根据文件类型设置过期时间
```
location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
    if (-f $request_filename) {
        expires 1h;
        break;
    }
}
```
### 禁止访问某个目录
```
location ~* \.(txt|doc)${
    root /data/www/wwwroot/linuxtone/test;
    deny all;
}
```

