---
title: "部署与发布"
description: "把站点发布到可访问的地址（Nginx 或 GitHub Pages）。"
---

本页用于记录部署方案与操作步骤（建议由负责部署的同学持续更新）。

## 方案 A：Linux + Nginx（虚拟机/云服务器）

**目标**：让站点可以通过外网访问（IP/域名 + 端口）。

建议步骤：

1. 安装 Nginx
2. 配置防火墙/安全组放行 80/443（或自定义端口）
3. 生成静态文件：`hugo -D`（输出到 `public/`）
4. 上传 `public/` 到服务器（例如 `/var/www/site/`）
5. Nginx 配置 `root` 指向站点目录，重载 Nginx

> 具体命令与配置片段请根据你们的服务器系统补充到这里（Ubuntu/CentOS 等）。

## 方案 B：GitHub Pages（可选）

如果你们希望免服务器部署，可以考虑 GitHub Pages。常见做法：

- 让 GitHub Pages 直接从仓库根目录或 `public/` 发布
- 或用 CI（GitHub Actions）自动构建并发布

后续如果你把 Pages 的发布方式定下来，我可以把配置补齐并帮你一键跑通。

