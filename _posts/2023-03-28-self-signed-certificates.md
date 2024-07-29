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

创建自签名证书，支持多泛域名和 IP，适用于在本地开发和测试。

## 生成证书

```bash
cat <<EOF | tee my.ini
[req]
default_bits = 4096
default_keyfile = server.pem
distinguished_name = req_distinguished_name
encrypt_key = no
default_md  = sha256
req_extensions = req_ext

[req_distinguished_name]
countryName = Country Name (2 letter code)
countryName_default = CN
commonName = Common Name (e.g. server FQDN or YOUR name)
commonName_default = my.dev
stateOrProvinceName = State or Province Name (full name)
stateOrProvinceName_default = Beijing
localityName = Locality Name (eg, city)
localityName_default = Beijing
organizationName = Organization Name (eg, company)
organizationName_default = Personal
organizationalUnitName = Organizational Unit Name (eg, section)
organizationalUnitName_default = Me
emailAddress = Email Address
emailAddress_default = me@my.dev

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = my.dev
DNS.2 = *.my.dev
IP.1 = 192.168.1.1
IP.2 = 192.168.1.2
IP.3 = 10.0.0.1
IP.4 = 10.0.0.2
EOF

mkdir -p output
openssl genrsa -out output/root-ca.key 4096
openssl req -sha256 -new -nodes -x509 -days 3650 -key output/root-ca.key -out output/root-ca.crt -subj "/CN=Personal Root CA/O=Personal/OU=Me"

openssl genrsa -out output/my.key 4096
openssl req -new -sha256 -key output/my.key -out output/my.csr -subj "/CN=my.dev/O=Personal/OU=Me"
openssl x509 -req -in output/my.csr -CA output/root-ca.crt -CAkey output/root-ca.key -CAcreateserial -days 3650 -extfile my.ini -extensions req_ext -out output/my.crt
```

{: file='generate-certificate.bash'}

> 格式转换
> ```bash
> # CRT 转成 PEM
> openssl x509 -outform PEM -in output/my.crt -out output/my.pem
>
> # CRT 转成 CER
> openssl x509 -outform der -in output/my.crt -out output/my.cer
>
> # CRT 转成 PFX
> openssl pkcs12 -export -in output/my.crt -inkey output/my.key -password pass:change@me -out output/my.pfx
>
> # CRT 转成 P7B
> openssl crl2pkcs7 -nocrl -certfile output/my.crt -out output/my.p7b
>
> # CRT 转成 P12
> openssl pkcs12 -export -in output/my.crt -inkey output/my.key -passin pass:change@me -name '*.my.dev' -chain -CAfile output/RootCA.crt -password pass:change@me -caname '*.my.dev' -out output/my.p12
> # 可选：查看 P12 证书
> #keytool -rfc -list -keystore output/my.p12 -storetype pkcs12
>
> # P12 转成 JKS
> keytool -importkeystore -srckeystore output/my.p12 -srcstoretype PKCS12 -deststoretype JKS -destkeystore output/my.jks
> keytool -importkeystore -srckeystore output/my.jks -destkeystore output/my.jks -deststoretype pkcs12
> ```
{: .prompt-tip }

## 服务器配置

### Nginx

- 引入证书和私匙：
```conf
ssl_certificate      /data/ssl/my.crt;
ssl_certificate_key  /data/ssl/my.key;
```
- 配置域名： 
服务端口添加上`ssl`，如`listen  443 ssl;`。

## 客户端配置

### Linux

- 将根证书拷贝到`/usr/local/share/ca-certificates`；
- 更新证书：`update-ca-certificates`。

### Mac OS

- 双击根证书，通过`钥匙串访问`打开并导入证书；
- 在`钥匙串访问`双击导入证书，设置<kbd>信任</kbd>-<kbd>使用此证书时:</kbd>始终信任。

### Windows

- 双击根证书，点击<kbd>安装证书</kbd>；
- 安装目录选择`受信任的根证书颁发机构`，按提示完成安装。
