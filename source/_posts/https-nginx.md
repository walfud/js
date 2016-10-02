---
title: let's https --- 基于 Let's Encrypt 实现 https (ubuntu 16.04 + nginx)
tag:
  - https
date: 2016/9/30
---

# 起源
http 是不安全的, 源于:
* 网络的通信线路是公用的. 你的数据同样会被传送到别人的网卡里
* http 是明文传输

在网上传输 http 相当于裸奔!

此外, https 相对于 http, 还有速度上的优势. 具体 https 的优点, 可以看看 [Google I/O 2016](https://www.youtube.com/watch?v=YMfW1bfyGSY).


# https 工作原理
![](/images/https_nginx/stack.png)

简单来说, 在左侧原始 http 的基础上, 右侧的 https 引入了一层 SSL, 在数据发送和接收的时候, 自动的进行加密/解密. 这样的加密数据在网络上传输, 即便被窃取到也不会有问题. 同时, 对于接收数据而言, 上层应用(http) 依然会收到被 SSL 层解析好的明文数据, 这对于已存在的 http 应用提供了 100% 的兼容性.


# https 实践
### 证书机构选择
当下有很多证书机构:
* 做证书起家的 [VeriSign](http://www.verisign.com/)
* 老牌的 [StartSsl](https://www.startssl.com/)
* https 的新星: [Let's Encrypt](https://letsencrypt.org/)
* 还有国内的 [沃通](https://www.wosign.com/)

经过一番调研和对比, 直接选择 Let's Encrypt 就好了. 免费, 快速, 方便自动化续签, 未来支持潜力更好...

### Let's Encrypt 工作流程
假设我们有:
* 一台装有 nginx 的 vps
* 一个域名(walfud.com), 这个域名已经指向了上述 vps

###### 申请证书
想要使用 https, 我们需要向 Let's Encrypt 证明你拥有该域名的控制权. 传统的证书申请机构是使用邮箱作为验证, 即: 发送一封邮件到你要签名的域名让你去确认. 而 Let's Encrypt 使用一种更加先进的方法, 叫做 [ACME](https://ietf-wg-acme.github.io/acme/). 这种方法的大致思路就是: 在你域名所指向的服务器上写入某些随机内容的文件, 如果发起 ACME 的服务器能够读成功, 就认为你拥有该域名的控制权. ACME 对比传统的邮件方式, 可以免去人工打开邮箱等过程, 从而实现自动化续签.

1. 在你的 vps 上下载官方推荐的工具: [certbot](https://certbot.eff.org/). 这里我们选择 _Nginx_ 和 _Ubuntu 16.04 (xenial)_. 如图
![](/images/https_nginx/certbot.png)
2. 下面的 _automated_ 页面中, 介绍了具体执行的方法. 我直接给出命令:
```shell
sudo apt-get install letsencrypt
letsencrypt certonly --agree-tos --email walfud@aliyun.com --webroot -w /usr/share/nginx/html -d walfud.com
```
其中,
  * `sudo apt-get install letsencrypt` 过程中, 可能 _install python..._ 这一步会卡很久, 请耐心等待, 它真的是很慢而已...
  * `walfud@aliyun.com` **替换为你的 email !!!**
  * `/usr/share/nginx/html` 是 _ACME Challenge_ 的路径(看后面的 [术语](#term))
  * `walfud.com` 这就是你要申请的域名. **这个域名一定要指向你的这台 vps!!!** (先 ping 一下看看结果是不是你主机的 ip)
3. 成功会出现: _Congratulations! Your certificate and chain have been saved at..._, 如图:
![](/images/https_nginx/congratulations.png)
_图中以 test.walfud.com 作为域名进行验证. 请以你本地输出为准_

检查 `/etc/letsencrypt/live/walfud.com/` 这个目录(可能需要 `sudo` 权限), 应该已经产生了如下几个文件 (完整解释请看这里: [certbot 的 webroot 插件文档](https://certbot.eff.org/docs/using.html#id25)):
  * cert.pem: 你不用关心 (这个实际上是服务器证书文件)
  * chain.pem: 你不用关心 (这个实际上是... 自己看文档吧, 我没读懂. 貌似是个递归查找用的链式证书)
  * fullchain.pem: cert.pem + chain.pem 的合体. 需要配置到 nginx 配置文件中的 `ssl_certificate`.
  * privkey.pem: 私钥. 需要配置到 nginx 配置文件中的 `ssl_certificate_key`.

###### 配置 Nginx 使用证书
1. 打开 `/etc/nginx/default.conf`(如果没有则创建), 然后粘贴我的配置:
  ```
  server {
      listen 80;
      # listen [::]:80;
      server_name walfud.com;

      # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
      return 301 https://$host$request_uri;
  }

  server {
      listen 443 ssl http2;
      # listen [::]:443 ssl http2;
      server_name walfud.com;

      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }

      #error_page  404              /404.html;

      # redirect server error pages to the static page /50x.html
      #
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }

      # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
      ssl_certificate /etc/letsencrypt/live/walfud.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/walfud.com/privkey.pem;
      ssl_session_timeout 1d;
      ssl_session_cache shared:SSL:50m;
      ssl_session_tickets off;

      # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
      # ssl_dhparam /etc/ssl/certs/dhparam.pem;

      # intermediate configuration. tweak to your needs.
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
      ssl_prefer_server_ciphers on;

      # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
      add_header Strict-Transport-Security max-age=15768000;

      # OCSP Stapling ---
      # fetch OCSP records from URL in ssl_certificate and cache them
      ssl_stapling on;
      ssl_stapling_verify on;

      ## verify chain of trust of OCSP response using Root CA and Intermediate certs
      # ssl_trusted_certificate /etc/letsencrypt/live/walfud.com/root_ca_cert_plus_intermediates;

      # resolver 8.8.8.8 8.8.4.4 valid=300s;
      # resolver_timeout 5s;
  }
  ```
  **别忘了把 `walfud.com` 都替换为你的域名!!!**
2. 重启 Nginx: `nginx -s reload`. 好了, [打开浏览器, 试试吧](https://walfud.com).
![](/images/https_nginx/https.png)

###### 证书续签
你买域名的时候, 都会有一个期限, 一年, 两年, 五年或者十年. 域名不能永远属于你, 那么证书也需要一个过期时间.

Let's Encrypt 家的证书有效期是 90 天, 所以我们为了保证证书的有效性, 就需要定期的 renew(续签) 证书. 一般来说, 大家都是用 `cron` 系统服务来定期的执行 `renew`:
* `crontab -e` 打开编辑界面
* 写入每天夜里凌晨 2 点重新续签:
  ```
  # 每天夜里凌晨 2 点续签:
  * 2 * * * letsencrypt renew

  # 重启 nginx 以使证书生效
  * 3 * * * nginx -s reload
  ```
这样, 每天都会尝试续签一次证书. 你不用担心什么, 如果证书没过期, 这个命令什么也不做. 如果证书过期, 则会帮你自动刷新证书.


# FAQ
<p>
Q: 证书是针对域名签发的还是 IP 签发的? <br/>
A: 是针对域名签发的. 我曾经在 123.206.49.60 这台机器上申请了 test.walfud.com 的证书, 然后把证书放到 123.56.81.162 机器上, 依然可以使用.
</p>

# <a name="term">术语解释</a>
* 对称密码: 通信两端使用同一密码进行加密/解密. 这个密码就称对称密码. 参考 [对称密码与公钥密码](http://js.walfud.com/cryptology#symmetric)
* 公钥 / 私钥: 通信两端使用非对称密码交换数据. 参考 [对称密码与公钥密码](http://js.walfud.com/cryptology#asymmetric)
* 签名: 发送者证明所发的内容确实来自于自己的凭证. 参考 [密码学 ABC](http://android.walfud.com/%E5%AF%86%E7%A0%81%E5%AD%A6-abc/)
* 证书: 其实是数字签名的一种应用. 即: 公钥 + 公钥的数字签名. 用以证明该公钥是可信的. 参考 [<<图解密码技术>>](https://item.jd.com/11942019.html)
* 续签: 每个证书都有有效期, 过了有效期需要重新向颁发机构申请.
* ACME Challenge: 即: ACME 服务器生成一个随机序列, 然后尝试读取某个域名下的该序列. 如果该序列能够被读取, 则认为你拥有该域名的控制权.


# Refs
[对称密码与公钥密码](http://js.walfud.com/cryptology)

[密码学 ABC](http://android.walfud.com/%E5%AF%86%E7%A0%81%E5%AD%A6-abc/) (可惜配图没了, 但不影响阅读)

[Let's Encrypt 给网站加 HTTPS 完全指南](https://ksmx.me/letsencrypt-ssl-https/)

[在 Nginx 上使用 Let’s Encrypt 加密(HTTPS)你的网站](http://www.appinn.com/use-letsencrypt-with-nginx/)

[Let's Encrypt Getting Started](https://letsencrypt.org/getting-started/)

[Let's Encrypt How It Works](https://letsencrypt.org/how-it-works/)

[certbot Retrive & Renew Certificates Automatically](https://certbot.eff.org/#ubuntuxenial-nginx)

[certbot 的 webroot 插件文档](https://certbot.eff.org/docs/using.html#webroot)

[How To Secure Nginx with Let's Encrypt on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)

[How To Use Cron To Automate Tasks On a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-on-a-vps)

帮你生成 Nginx 配置的最佳实践: [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)
