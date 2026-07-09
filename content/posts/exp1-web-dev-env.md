---
title: "实验1：Web开发环境搭建"
date: 2026-07-09
draft: false
---

## 实验目标

安装 Web 开发环境（VS Code、Git、Hugo），构建一个静态站点，了解和实践 Web 服务器、软件包及管理器等工具和概念。

## 实践过程

1. 下载并安装 VS Code，安装 Markdown 插件
2. 下载并安装 Git，配置用户信息
3. 下载 Hugo Extended 版本，解压并配置环境变量
4. 使用 `hugo new site` 创建站点，下载 Ananke 主题
5. 本地预览，确认站点正常运行

## 遇到的问题与解决方案

**问题1：Hugo 安装后终端提示 "hugo 不是内部或外部命令"**

原因：没有将 Hugo 的安装路径添加到系统环境变量 Path 中。

解决方案：将 `C:\hugo\` 路径添加到系统环境变量 Path，然后重新打开终端即可生效。

**问题2：PowerShell 中 git submodule 命令报错**

原因：命令格式错误，`git` 和 `themes/ananke` 之间缺少空格。

解决方案：确保命令格式正确：`git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke`

**问题3：hugo.toml 文件编码问题导致 Hugo 无法加载配置**

原因：使用 PowerShell echo 命令生成的文件带有 BOM 头，Hugo 无法识别。

解决方案：在 VS Code 中手动创建 hugo.toml，并保存为 UTF-8 无 BOM 格式。

## 实验截图


![站点预览](/images/screenshow1.png)
![Git提交记录](/images/screenshot2.png)