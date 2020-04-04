---
title: 记录个blog搭建的流水账
date: 2020-04-04 16:48:52
tags:
  - blog
  - 博客
  - 流水账
  - 阿里云
  - ECS
  - nginx
  - hexo
---


在阿里云重新搭建blog. 打算先用 node + hexo 先搭起来一个轻量的静态博客先用着。

# 简单部署一下 hexo

1. 安装 hexo-cli
2. 在 ~/hexoblog 里初始化了blog项目 mrraindrop.cn
3. hexo -g 生成 public 目录.

# 通过 ssh 登录 ECS 实例

首先，需要在 ECS 实例管理界面的网络与安全里，设置密钥对，可以新建一个新的秘钥对。建好以后会立即下载一个 pem 证书文件。将证书文件妥善保管。

接下来需要通过 pem 生成 .ppk 私钥文件。Window 下可以通过 PuttyGen 将 pem 生成 .ppk。将 Type of key to generate 设置为 RSA，然后点击 Load. 加载完生成的 pem 文件，然后点 File -> Save private key 就可以将私钥保存为 .ppk 文件了。

以后在用 putty 连接的时候，都可以使用这个 ssh key 了。只需要在 Connection -> Auth -> Browser 里选取该 .ppk 文件，然后在 Session 里对应的连接里，将 Connection type 选择为 SSH，连接的时候就会使用对应的 ssh key 了。

如果不想每次输入 login as: root，在连接 ip 前加上 root@  就可以了。

# 在 vscode 里编辑 & 发布

在 vscode 里直接打开 ECS 里的 blog 目录，写完以后直接在 vscode 命令行里面 hexo g 提交，然后立刻就能在网站看到效果？你只需要这个插件，就能搞定上述需求：

> Remote - SSH

然后在本地的 `c:\Users\xxx\.ssh` 里添加一个 config 文件，将远程连接的 pem 信息添加进去：

```
Host ECS
User [yourLoginUser]
Hostname [yourIp]
IdentityFile C:\Users\xxx\.ssh\aliyun***.pem
```

# 安装配置 nginx

```
yum install nginx -y
```

启动服务：

```
systemctl start nginx.service
```

打开 /etc/nginx/nginx.conf，备份一下，然后修改 server 里面的配置，比如 server_name 设置为绑定的域名，root 设置为 hexo 目录下的 public 目录的路径。

然后测试一下更新的 conf 是否有用：

```
nginx -t
```

测试成功，则

```
nginx -s reload
```

或者

```
nginx -s stop
nginx -c /etc/nginx/nginx.conf
```

在网页端访问 403 了？

1. 将 hexo public 目录设置权限为 777： chmod -R hexoRoot
2. 将 nginx config 里的 user 设置为 hexo 文件夹的 owner.

# 安装 https

在阿里云里购买免费的 ssl 证书，下载到本地，然后 copy 到 nginx/cert 目录里。
在 nginx/nginx.conf 配置的 https 配置里，将 cert 和 key 的文件路径指向刚才 copy 到 cert 目录里的文件.

## 强制 https 访问

打开 80 的访问，listen 80 和 server_name 不变，然后将其他部分注释，新写一行：

```
rewrite ^(.*)$ https://$host$1 permanent;
```

这样 http 访问就会重定向到 https 了.
