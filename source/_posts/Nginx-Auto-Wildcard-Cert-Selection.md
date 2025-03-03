---
# title: Nginx Auto Wildcard Cert Selection
title: Nginx 自动选择通配符证书
date: 2025-03-02 22:45:00
tags: 运维
---

## TL;DR

{% codeblock "/etc/nginx/conf.d/ssl_map.conf" lang:nginx %}
map $ssl_server_name $MatchedCert {
    default '/etc/nginx/ssl/localhost_P256/localhost.crt';
    ~*^([^.]+)\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/fullchain.cer';
    ~*^([^.]+)\.r\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/fullchain.cer';
}
map $ssl_server_name $MatchedKey {
    default '/etc/nginx/ssl/localhost_P256/localhost.key';
    ~*^([^.]+)\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/private.key';
    ~*^([^.]+)\.r\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/private.key';
}
{% endcodeblock %}

{% codeblock "/etc/nginx/conf.d/ssl.conf" lang:nginx %}
server {
    include /etc/nginx/conf.d/ssl_map.conf;
    server_name example.xiaoxuan010.top;
    ...
}
{% endcodeblock %}

## 问题

在一个 Nginx 实例中，持有多个域名通配符证书，传统做法需要为每个网站的 `server` 块配置证书路径，手动选择证书：

{% box child:codeblock color:yellow %}

```nginx
server {
    server_name example.xiaoxuan010.top;
    ssl_certificate '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/fullchain.cer';
    ssl_certificate_key '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/private.key';
    ...
}

server {
    server_name example.r.xiaoxuan010.top;
    ssl_certificate '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/fullchain.cer';
    ssl_certificate_key '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/private.key';
    ...
}
```

{% endbox %}

每次新增网站，都要手动配置长长的证书路径，非常繁琐。

## 解决方案

在 Nginx 配置目录中，创建一个 `ssl_map.conf` 文件，用于配置域名和证书的映射关系：

{% codeblock "/etc/nginx/conf.d/ssl_map.conf" lang:nginx %}
map $ssl_server_name $MatchedCert {
    default '/etc/nginx/ssl/localhost_P256/localhost.crt';
    ~*^([^.]+)\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/fullchain.cer';
    ~*^([^.]+)\.r\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/fullchain.cer';
}
map $ssl_server_name $MatchedKey {
    default '/etc/nginx/ssl/localhost_P256/localhost.key';
    ~*^([^.]+)\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.xiaoxuan010.top_xiaoxuan010.top_P256/private.key';
    ~*^([^.]+)\.r\.xiaoxuan010\.top$ '/etc/nginx/ssl/#.r.xiaoxuan010.top_r.xiaoxuan010.top_P256/private.key';
}
{% endcodeblock %}

这段代码创建了两个 [map](https://nginx.org/en/docs/http/ngx_http_map_module.html) 块，分别用于匹配证书路径和私钥路径。[$ssl_server_name](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#:~:text=established%20SSL%20connection%3B-,%24ssl_server_name,-returns%20the%20server) 是 Nginx 内置变量，表示当前请求的域名。`$MatchedCert` 和 `$MatchedKey` 这两个变量的值取决于 `$ssl_server_name` 的值，会根据正则表达式匹配域名，选择对应的证书和私钥路径。

之后，在每个网站配置的 `server` 块中，引入 `ssl_map.conf` 文件，并设置 `server_name`：

{% codeblock "/etc/nginx/conf.d/ssl.conf" lang:nginx %}
server {
    include /etc/nginx/conf.d/ssl_map.conf;
    server_name example.xiaoxuan010.top;
    ...
}
{% endcodeblock %}

大功告成，以后新增域名只需要在 `ssl_map.conf` 中添加两行映射关系即可，新增网站也不需要再配置证书路径。
