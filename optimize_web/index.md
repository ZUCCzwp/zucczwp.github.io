# 网站优化


### 定义
网站地图（sitemap.xml）是一个网站的概览。搜索引擎在此文件中得到网站上存在的可抓取的网页，提交sitemap有利于搜索引擎的收录。

网站地图是一个网站的缩影，包含网站的内容地址，是根据网站的结构、框架、内容，生成的导航文件。网站地图分为三种文件格式：xml格式、html格式以及txt格式。xml格式和txt格式一般用于搜索引擎，为搜索引擎蜘蛛程序提供便利的入口到你的网站所有网页；html格式网站地图可以作为一个网页展示给访客，方便用户查看网站内容。

### 生成方法
Sitemap 0.90 是依据创意公用授权-相同方式共享的条款提供的，并被广泛采用，受 Google、Yahoo! 和 Microsoft 在内的众多厂商的支持。[网址](https://www. sitemaps.org/zh_CN/faq.html)
```
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"     
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"     
 xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9            
 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">
 <url> 
     <loc>http://www.zwp.com/</loc>  
     <changefreq>daily</changefreq>
 </url>
 <url> 
     <loc>http://www.zwp.com/wyblog?catid=43</loc>  
     <changefreq>daily</changefreq>
 </url>
 <url> 
     <loc>http://www.zwp.com/wyblog/?catid=167</loc>  
     <changefreq>daily</changefreq>
 </url>
 <url> 
     <loc>http://www.zwp.com/wyblog/?catid=170</loc>  
     <changefreq>daily</changefreq>
 </url> 
</urlset>
```

### 提交步骤
1. 生成网站地图：
* 网页版：http://www.xml-sitemaps.com/
* 客户端：http://cn.sitemapx.com/

change frequency:指的是频率，地图的自动更新频率，默认每天(daily);

last modification:是网站地图最后修改时间，默认使用服务器的响应(Use server's response);

priority:权重-可自动计算。

点击start开始生产。自动跳转到生成页面，稍等一段时间便可生成（时间和网站内容多少有关）。

它提供多种格式的网站地图文件下载(xml、xml.gz、ror.xml、html等)，看你所提交的搜索引擎需要哪种格式的地图文件，就下载哪一个，或者直接打包全下载。

2. 下载生成的地图文件sitemap.xml并上传至网站根目录
3. 到站长平台提交网站地图。

{{<admonition >}}
各搜索引擎推荐网站地图格式：

Google：建议使用xml格式的网站地图
[Google提交地址](https://www.google.com/webmasters/tools/dashboard?hl=zh-CN)

Yahoo： 建议使用Txt格式的网站地图
[Yahoo提交地址](http://sitemap.cn.yahoo.com/?require_login=1)

Baidu：建议使用robots.txt提交html格式的网站地图
[Baidu提交地址](http://www.baidu.com/search/url_submit.html)

在robots.txt爬虫协议中定义（Allow: /sitemap.hxml）robots.txt文档是表式允许蜘蛛网访问网站地图。爬虫协议写法参见：网站优化之robots.txt爬虫协议的写法。
{{</admonition >}}
