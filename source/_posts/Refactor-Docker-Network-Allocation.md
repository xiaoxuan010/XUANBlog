---
# title: Refactor Docker Network Allocation
title: 优化 Docker 子网划分方式
excerpt: 修改 Docker （尤其是 Docker Compose）默认的网络分配方式，避免浪费子网容量。
date: 2026-01-29 16:00:00
tags:
  - DevOps
  - Docker
---

## 问题引入：为什么要重新划分子网

我们在 Docker 中启动下面这个最简单的 Compose：

```yaml compose.yml
services:
  hello:
    image: hello-world
```

通过 `docker network inspect hello-world_default` 我们发现 Docker 自动为我们创建了一个 `172.xx.0.0/16` 的网络，这对于大部分简单的容器来说都太大了。当启动的 Compose 较多时，占用了很多宝贵的 [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) 私有地址空间。

## 配置目标

本文配置的目标是：

- 默认的 `bridge` 网络使用 `172.16.0.1/16` 网络。
- 其他网络默认使用 `172.18.0.0/16` 空间的网络，且每个网络大小默认为 `/24`。

## 操作步骤

### 修改 Docker 配置

编辑 `/etc/docker/daemon.json`（不存在则创建）：

```json
{
  "bip": "172.16.0.1/16",
  "default-address-pools": [
    {"base": "172.18.0.0/16", "size": 24}
  ]
}
```

### 重启 Docker

```bash
systemctl restart docker
```

### 重建 Compose 项目

对每个 Compose 项目：

```bash
cd /path/to/compose/project
docker compose down
docker compose up -d
```

### 清理旧网络（可选）

```bash
docker network prune -f
```

## 总结

经过以上步骤，我们成功修改了 Docker 的默认网络分配方式。其中，使用 `/24` 子网是因为这样能使点分十进制地址的第三段表示网络号，第四段表示主机号，更加直观。如果有其他需求，可根据实际需要调整 base 和 size 参数，详见[官方文档示例](https://docs.docker.com/reference/cli/dockerd/#:~:text=%22default%2Daddress%2Dpools%22)。
