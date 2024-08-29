---
layout: post
title: 使用 Docker 搭建 Virtual Studio Code 远程开发环境
date: 2023-03-31 11:19 +0000
categories:
  - 技术
tags:
  - vscode
  - remote
  - ide
---

Virtual Studio Code 支持服务器安装了，通过自带的 CLI 工具，可快速在服务器部署并运行 IDE 远程服务，自带穿透功能实现任意地方远程访问。


## 创建 Docker 镜像

```Dockerfile
FROM ubuntu as builder
WORKDIR /tmp/build
RUN true \
  && apt-get -qq update \
  && apt-get -qq install curl -y > /dev/null \
  && curl -sSL "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64" | tar xzf -


FROM ubuntu AS release
COPY --from=builder /tmp/build/code /usr/local/bin/code
RUN true \
  && apt-get -qq update \
  && apt-get -qq install ca-certificates -y > /dev/null \
  && rm -rf /root/.cache /var/lib/apt/lists/* /var/cache/apt
ENTRYPOINT code
CMD tunnel user login --provider microsoft --accept-server-license-terms
```
{: file='./images/vscode/Dockerfile'}

## 运行服务

```yaml
name: svc
services:
  vscode:
    image: vscode
    container_name: vscode
    restart: unless-stopped
```
{: file='docker-compose.yaml'}

```shell
docker-compose up -d vscode
```
- 按提示访问<https://github.com/login/device>，并输入授权码；
- 访问<https://vscode.dev/tunnel/vscode>即可。

> 远程开发环境是自带终端功能的，权限很大，可做的事很多，存在很大风险，使用时请确保安全。
{: .prompt-danger }
