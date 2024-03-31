[TOC]

# 使用 certbot 快速申请 Let's Encrypt 证书

>申请 https 的可信证书可以提高自己 website 的安全性，之前因为觉得都是静态内容所以没必要用 https，主要就是嫌申请 https 证书比较麻烦. 最近折腾了一下发现，使用命令行的 certbot 可以非常方便的获取 Let’s encrypt 的免费证书.

## Certbot 简介
Github 地址：https://github.com/certbot/certbot
本质上来说，certbot 就是一个 ACME client，这也是 Let’s Encrypt 官网推荐的签发证书的方式，适用于对自己的 domain 具有 shell 访问能力的情况，使用所谓的 ACME 协议来自动化的签发证书，很大程度上简化了证书签发的步骤，

## 准备工作
HTTPS 的证书基于 domain 签发，所以事先需要申请一个 domain，例如：example.me.
一台公网能够访问的 VPS，将 example.me 解析为它的 IP.
## 一键运行
可以从 github 来安装 certbot，但文档中也给出了直接用 docker 启动的方法，还是非常方便的


```
sudo docker run -it --rm --name certbot \
-v "/home/ubuntu/docker/certbot/etc/letsencrypt:/etc/letsencrypt" \
-v "/home/ubuntu/docker/certbot/var/lib/letsencrypt:/var/lib/letsencrypt" \
-p 80:80 \
certbot/certbot certonly

```

其中 certonly 是获取 cert 的命令，映射 80 的目的是为了一会 ACME server 来访问我们的 VPS 验证所有权.


## 填写信息
如果不加任何参数运行 certonly 的话就会进入到下面的交互式界面：

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

How would you like to authenticate with the ACME CA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Runs an HTTP server locally which serves the necessary validation files under
the /.well-known/acme-challenge/ request path. Suitable if there is no HTTP
server already running. HTTP challenge only (wildcards not supported).
(standalone)
2: Saves the necessary validation files to a .well-known/acme-challenge/
directory within the nominated webroot path. A seperate HTTP server must be
running and serving files from the webroot path. HTTP challenge only (wildcards
not supported). (webroot)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):

```

选项 1 是让 certbot 自己运行 HTTP server 来通过验证，而选项 2 则是我们自己需要放置 .well-known/acme-challenge/ 目录的内容来通过验证.
如果不怕自己 VPS 的 web 服务中断的话，第 1 种方式还是比较方便的.

随后就是填写邮件信息和同意服务协议：
```
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): alice@bob.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Account registered.

 ```

接下来就可以输入自己需要签发证书的 domain 了，这里可以**用 , 或者空格** 隔开多个 domain，但不能使用通配符 *. 这时 Certbot 会在本地启动一个 http server，然后通知 ACME server 来访问我们 domain 对应的 website，以验证 domain 的所有权.

```
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): example.me
Requesting a certificate for example.me

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/example.me/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/example.me/privkey.pem
This certificate expires on xxxx-xx-xx.
These files will be updated when the certificate renews.

```

到这里就签发完毕了，在 host 的 /home/ubuntu/docker/certbot/etc/letsencrypt/live/example.me/ 目录下就生成证书 fullchain.pem 和 privkey.pem. 然后我们就可以在自己的 web server 配置 SSL 证书了.

## 更新
用这种方式生成的短期证书有效期是 90 天，在过期之后我们还需要对其进行更新（renew）操作，只需要将上面的命令 certonly 改为 renew 即可，该命令会自动更新 /home/ubuntu/docker/certbot/etc/letsencrypt/live/ 目录下有效期少于 30 天的证书.