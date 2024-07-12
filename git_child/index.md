# gitlab私有库


## 一、设置go env
设置私有仓库的git地址

```
go env -w GOPRIVATE="git.example.com"
```

## 二、配置git

{username}： 替换成gitlab账号名称

{token}： 替换成自己的 Personal Access Token

#### *token*生成：gitlab → Preferences → Access Tokens，填写完Token名称和过期时间，勾选好全部的权限，生成即可

使用命令行

```
git config --global url."git.weyland-x.com".insteadOf "https://{username}:{token}@git.example.com"
```

或者直接编辑`~/.gitconfig`

```
[url "git@git.example.com/"]    
    insteadof = https://{username}:{token}@git.example.com/
```

## 三、配置 .netrc
解决gitlab不能拉取子组库的问题

**linux/mac下**

在用户目录下编辑`~/.netrc`文件

```
machine git.example.com
login {Username}
password {Personal Access Token}
```

**windows下**

在用户目录下创建 _netrc 文件

```
machine git.example.com
login {Username}
password {Personal Access Token}
```

自行替换username和token，与git配置中的相同
