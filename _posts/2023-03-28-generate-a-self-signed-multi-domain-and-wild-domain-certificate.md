---
layout: post
title: 生成自签名的多泛域名证书
date: 2023-03-28 06:32 +0000
categories:
- 开发
- 服务
tags:
- openssl
- nginx
---

随着 HTTPS 的普及，很多场景需要通过 HTTPS 进行测试验证。这里提供的解决方案是自签名任意域名（包括泛域名）证书，优点是可以通过简单易记的域名一劳永逸解决域名+证书问题，缺点是只能内网使用。

## 生成证书

### 配置证书
```conf
[req]
default_bits        = 2048
default_keyfile     = server-key.pem
distinguished_name  = subject
req_extensions      = req_ext
x509_extensions     = x509_ext
string_mask         = utf8only

[subject]
countryName                 = Country Name (2 letter code)
countryName_default         = CN
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Guangdong
localityName                = Locality Name (eg, city)
localityName_default        = Shenzhen
organizationName            = Organization Name (eg, company)
organizationName_default    = Jinman,Lai
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_default          = laijinman
emailAddress                = Email Address
emailAddress_default        = admin@laijinman.com

[x509_ext]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, keyEncipherment
subjectAltName         = @alternate_names
nsComment              = "OpenSSL Generated Certificate"

[req_ext]
subjectKeyIdentifier = hash
basicConstraints     = CA:FALSE
keyUsage             = digitalSignature, keyEncipherment
subjectAltName       = @alternate_names
nsComment            = "OpenSSL Generated Certificate"

[alternate_names]
DNS.1 = laijinman.dev
DNS.2 = *.laijinman.dev
DNS.3 = laijinman.localhost
DNS.4 = *.laijinman.localhost
```
{: file='laijinman.cnf'}

### 创建证书

```bash
openssl req -config laijinman.cnf -new -sha256 -newkey rsa:2048 -nodes -keyout laijinman.key -x509 -days 3650 -out laijinman.crt
```
一路回车<kbd>Enter</kbd>。

## 服务器配置

### Nginx

```conf
ssl_certificate      /data/ssl/laijinman.crt;
ssl_certificate_key  /data/ssl/laijinman.key;
```
需要开通 HTTPS 访问的域名添加上`listen  443 ssl;`即可。

## 客户端配置

### Mac OS

- 通过浏览器访问域名，提示证书不安全。在浏览器查看并导出证书到本地；
- 双击证书，通过私匙串访问打开并导入证书；
- 在私匙串访问双击导入证书，设置<kbd>信任</kbd>-<kbd>使用此证书时:</kbd>始终信任。

### Windows

- 通过浏览器访问域名，提示证书不安全。在浏览器查看并导出证书到本地；
- 双击证书，点击<kbd>安装证书</kbd>，按提示进行即可。
