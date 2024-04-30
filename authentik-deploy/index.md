# Authentik安装



### Docker安装
***MacOs***
1. 下载 docker composer文件
```
curl -O https://goauthentik.io/docker-compose.yml
```
2. 生成 password和 secret key到.env文件
```
echo "PG_PASS=$(openssl rand -base64 36)" >> .env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 36)" >> .env
```
3. 如果只要开启错误日志,运行
```
echo "AUTHENTIK_ERROR_REPORTING__ENABLED=true" >> .env
```
4. 邮件配置（可选不推荐）
```
# SMTP Host Emails are sent to
AUTHENTIK_EMAIL__HOST=localhost
AUTHENTIK_EMAIL__PORT=25
# Optionally authenticate (don't add quotation marks to your password)
AUTHENTIK_EMAIL__USERNAME=
AUTHENTIK_EMAIL__PASSWORD=
# Use StartTLS
AUTHENTIK_EMAIL__USE_TLS=false
# Use SSL
AUTHENTIK_EMAIL__USE_SSL=false
AUTHENTIK_EMAIL__TIMEOUT=10
# Email address authentik will send from, should have a correct @domain
AUTHENTIK_EMAIL__FROM=authentik@localhost
```
5. 配置端口号
```
COMPOSE_PORT_HTTP=80
COMPOSE_PORT_HTTPS=443
```
6. 开启
```
docker compose pull
docker compose up -d
```
7. 初始化配置
```
http://<your server's IP or hostname>/if/flow/initial-setup/
```

### 二进制安装
1. 确定 linux系统的架构
```
uname -m 
```
如果显示x86_64则是 amd架构 不是则是 arm,也可通过 arch命令得到
2. 前期准备
##### 安装依赖 #####
+ 安装 lib库
```
sudo yum update -y && sudo yum upgrade -y
sudo yum install -y curl wget git gcc gcc-c++ sqlite-devel readline-devel ncurses-devel 
openssl tk-devel gdbm-d evel db4-devel xz-devel make glibc-devel bzip2-devel pkgconfig 
libffi-devel libpcap-devel zlib-devel xmlsec1 xmlsec1-openssl libmaxminddb postgresql-devel
```
确定 devel库是否开启
+ 安装 python(要求版本 v3.12+)
```
wget https://www.python.org/ftp/python/3.12.2/Python-3.12.2.tgz
tar xzf Python-3.12.2.tgz 
cd Python-3.12.2
./configure --enable-optimizations
sudo make altinstall
rm -rf Python-3.12.2.tgz Python-3.12.2
```
+ 安装 node(要求版本 v18+)
```
wget https://nodejs.org/dist/latest-v21.x/node-v21.7.1-linux-x64.tar.xz 
tar xf node-v21.7.1-linux-x64.tar.xz 
```
配置环境变量
+ 安装 go(要求 v1.22+)
```
wget https://golang.org/dl/go1.22.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz
rm -rf go1.22.1.linux-amd64.tar.gz
vim ~/.profile 
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
source ~/.profile 
```
+ 安装 pip
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
rm -rf get-pip.py
```
+ 安装 py virtualenv
```
pip install virtualenv black ruff codespell bandit poetry pycparser psycopg2 xmlsec1
``` 
无网模式需要设置pypi代理
+ 安装 golangci-lint
#### 下载https://github.com/golangci/golangci-lint/releases对应 rpm ####
```
sudo rpm -ivh golangci-lint-1.57.2-linux-386.rpm
```
+ 安装 website、 web和 python依赖
```
cd /opt/authentik
make install
```
+ 安装 postgreSql
#### 下载路径：https://www.postgresql.org/ftp/source/v16.2/ ####
```
tar -zxvf postgresql-16.2.tar.gz
cd postgresql-16.2 
./configure
make & make install
```
+ 安装 redis
```
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar xzvf redis-4.0.8.tar.gz
cd redis-4.0.8
make
cd src
make install PREFIX=/usr/local/redis
cd ../
mkdir /usr/local/redis/etc
mv redis.conf /usr/local/redis/etc
vi /usr/local/redis/etc/redis.conf //将daemonize no改成daemonize yes
vi /etc/rc.local 
//在里面添加内容：/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
cp /usr/local/redis/bin/redis-server /usr/local/bin/
cp /usr/local/redis/bin/redis-cli /usr/local/bin/
```
+ 设置环境变量PATH:
```
export PATH="/home/admin/opt/authentik/.venv/bin":$PATH
export PATH="/home/admin/opt/authentik/lifecycle":$PATH
```
+ 启动 authentik服务
ak命令为 lifecycle目录下的可执行文件
```
ak server
ak worker
```
### 使用 ###
1. 初始化管理员账号密码

打开初始化页面 https://{{your host}}/if/flow/initial-setup/，设置邮箱密码
2. 访问管理后台

打开管理后台 https://{{your host}}/if/admin/#/administration/overview



