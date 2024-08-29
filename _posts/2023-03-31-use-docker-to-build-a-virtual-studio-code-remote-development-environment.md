---
layout: post
title: 使用 Docker 搭建 Virtual Studio Code 远程开发环境
date: 2023-03-31 11:19 +0000
categories:
- 开发
- 效率
tags:
- vscode
- remote
- ide
---

Virtual Studio Code 支持服务器安装了，通过自带的 CLI 工具，可快速在服务器部署并运行 IDE 远程服务，自带穿透功能实现任意地方远程访问。


## 创建 Docker 镜像

```Dockerfile
FROM ubuntu as downloader
WORKDIR /tmp/download
ADD https://az764295.vo.msecnd.net/stable/7f329fe6c66b0f86ae1574c2911b681ad5a45d63/vscode_cli_alpine_x64_cli.tar.gz code.tar.gz


FROM ubuntu as builder
COPY --from=downloader /tmp/download /tmp/download
WORKDIR /opt/vscode-cli/bin
RUN tar xf /tmp/download/code.tar.gz


FROM ubuntu AS release
COPY --from=builder /opt /opt
RUN apt-get update || true \
  && apt-get install ca-certificates -y \
  # clean
  && rm -rf /root/.cache /var/lib/apt/lists/* /var/cache/apt
ENTRYPOINT ["/opt/vscode-cli/bin/code", "tunnel"]
CMD ["--accept-server-license-terms"]
```
{: file='./images/vscode/Dockerfile'}

## 运行服务

```yaml
version: "3"

services:
  vscode:
    build:          ./images/vscode
    image:          docker.laijinman.dev/vscode-1.77.0:v1
    container_name: vscode
    hostname:       vscode
    restart:        unless-stopped
    volumes:
    - /root:/root
```
{: file='docker-compose.yaml'}

```shell
docker-compose up vscode
```
- 按提示访问<https://github.com/login/device>，并输入授权码；
- 访问<https://vscode.dev/tunnel/vscode>即可。

> 远程开发环境是自带终端功能的，权限很大，可做的事很多，存在很大风险，使用时请确保安全。
{: .prompt-danger }
