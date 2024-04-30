# Linux常用命令



### 删除多行
1.首先要显示对应的行数
```
:set nu
```
2.Esc退出
```
190,6233d
```
即[190 , 6233]都删除掉
### 清空文件
```
> log.txt
```
### 显示发行版本信息
```
lsb_release
-v 显示版本信息。
-i 显示发行版的id。
-d 显示该发行版的描述信息。
-r 显示当前系统是发行版的具体版本号。
-c 发行版代号。
-a 显示上面的所有信息。
-h 显示帮助信息。
```
### 查看文件时间
**Stat**
```
stat install.log
```
**Ls**
* modification time（mtime，修改时间）：当该文件的“内容数据”更改时，就会更新这个时间。内容数据指的是文件的内容，而不是文件的属性。
* status time（ctime，状态时间）：当该文件的”状态（status）”改变时，就会更新这个时间，举例来说，更改了权限与属性，就会更新这个时间。
* access time（atime，存取时间）：当“取用文件内容”时，就会更新这个读取时间。举例来说，使用cat去读取 ~/.bashrc，就会更新atime了。
```
ls -l --time=ctime install.log
```
### 修改文件时间
**Touch**
1. 同时修改文件的修改时间和访问时间
```
touch -d "2010-05-31 08:10:30" install.log
```
2. 只修改文件的修改时间
```
touch -m -d "2010-05-31 08:10:30" install.log
```
3. 只修改文件的访问时间
```
touch -a -d "2010-05-31 08:10:30" install.log
```
### 开启 devel库
***Linux***
```
vim /etc/yum.repos.d/rocky-devel.repo //确认是否开启
yum grouplist
yum group install '开发工具'
```
***Ubuntu/Debian***
```
sudo apt update
sudo apt install build-essential
```
### 查看进程
```
lsof -i:端口号
netstat -nap | grep 端口号
```
### 查看磁盘使用情况
```
du -f
```
### 抓包
```
tcpdump -D //列出可用的网卡列表
tcpdump -i eth0 //设置抓取的网卡名
tcpdump -i eth0 -w debug.cap //把捕获的包数据写入到文件中
tcpdump -A //以 ASCII 码方式显示每一个数据包
tcpdump -c 10 //设置抓取到多少个包后退出
tcpdump -w data.pcap port 80//将包数据写入文件 指定监听端口为 80
```

