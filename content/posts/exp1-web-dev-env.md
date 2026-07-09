---
title: "实验1：Web开发环境搭建"
date: 2026-07-09
draft: false
---

## 实验目标

本次实验的目标是完成 Web 开发环境的基本搭建，并能够运行一个 Hugo 静态网站。通过本次实验，需要熟悉 VS Code、Git、Hugo 等工具的安装和使用，理解本地预览、静态文件生成、代码仓库管理以及后续部署之间的关系。

对应评分表中的内容主要包括：

| 考核项目 | 本实验对应内容 |
|---|---|
| 项目与代码库配置 | 创建项目目录，配置 Git，整理 Hugo 项目结构 |
| Web 运行环境 | 安装 Hugo，运行本地开发服务器，生成静态网页 |
| Web 站点基本功能 | 能够访问首页、实验记录、任务问题、部署说明等页面 |

## 实验环境

| 项目 | 说明 |
|---|---|
| 操作系统 | Windows 11 |
| 编辑器 | Visual Studio Code |
| 版本管理工具 | Git |
| 静态网站生成器 | Hugo Extended |
| 网站主题 | Ananke |
| 本地预览地址 | `http://localhost:1313/` |

## 实践过程

### 1. 准备项目目录

先在本地创建课程网站项目目录，用于保存 Hugo 的配置文件、内容文件、静态资源和生成后的网页文件。项目目录中主要包含以下内容：

| 目录或文件 | 作用 |
|---|---|
| `hugo.toml` | 网站基础配置，包括标题、主题、菜单和样式文件 |
| `content/` | 网站正文内容，例如首页、实验记录、任务问题和部署说明 |
| `static/` | 静态资源目录，用于存放图片、CSS 等文件 |
| `themes/` | Hugo 主题目录 |
| `public/` | 执行构建后生成的静态网页文件 |

### 2. 安装和配置 Git

安装 Git 后，用于管理项目代码和后续提交到 GitHub 仓库。常用检查命令如下：

```powershell
git --version
git status
```

如果是第一次使用 Git，需要配置用户名和邮箱，便于提交记录中显示作者信息：

```powershell
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的邮箱"
```

### 3. 安装 Hugo 并运行本地服务

下载 Hugo Extended 版本后，将 `hugo.exe` 所在目录加入系统环境变量 Path。配置完成后重新打开 PowerShell，检查 Hugo 是否可用：

```powershell
hugo version
```

进入项目目录后运行本地预览服务：

```powershell
hugo server -D
```

浏览器访问 `http://localhost:1313/`，如果能看到网站首页，说明本地 Web 运行环境已经基本搭建成功。

### 4. 配置主题和页面内容

本网站使用 Ananke 主题，在 `hugo.toml` 中配置主题名称、网站标题、导航菜单和自定义样式文件。内容页面主要放在 `content/` 目录下，例如：

```text
content/_index.md
content/posts/exp1-web-dev-env.md
content/team/index.md
content/deploy/index.md
content/report/index.md
```

页面内容使用 Markdown 编写，便于小组成员分工维护，也方便后续把实验过程整理进报告。

### 5. 生成静态网页

本地确认页面正常后，执行构建命令生成 `public/` 目录：

```powershell
hugo -D
```

`public/` 中的内容就是后续可以部署到 Nginx、GitHub Pages 或其他 Web 服务器上的静态网站文件。

## 遇到的问题与解决方案

### 问题1：终端提示 `hugo` 不是内部或外部命令

原因：Hugo 已下载，但 `hugo.exe` 所在目录没有加入系统环境变量 Path，PowerShell 找不到这个命令。

解决办法：把 Hugo 解压目录加入系统 Path，或者在命令中直接使用 `hugo.exe` 的完整路径。修改环境变量后需要重新打开终端。

### 问题2：下载 GitHub ZIP 后主题目录为空

原因：Ananke 主题可能是以 Git submodule 的形式加入项目。直接下载 ZIP 时，子模块内容不会自动下载完整。

解决办法：重新补全主题目录，可以执行：

```powershell
git submodule update --init --recursive
```

如果不会使用子模块，也可以手动把 Ananke 主题下载或克隆到 `themes/ananke` 目录。

### 问题3：本地能打开页面，但样式没有变化

原因：自定义 CSS 没有被 Hugo 正确加载，或者浏览器缓存了旧页面。

解决办法：检查 `hugo.toml` 中是否配置了自定义 CSS，确认 `http://localhost:1313/css/custom.css` 能访问。修改样式后重启 `hugo server -D`，并在浏览器按 `Ctrl + F5` 强制刷新。

### 问题4：图片或截图在网页中显示失败

原因：Markdown 中使用了本机绝对路径，例如 `C:\Users\...`，这种路径部署到网站后无法访问。

解决办法：把截图统一放到 `static/images/`，页面中使用 `/images/图片名.png` 引用。这样本地预览和服务器部署时路径都比较稳定。

## 运行验证

本地验证时主要检查以下内容：

| 检查项 | 结果 |
|---|---|
| `hugo server -D` 能正常启动 | 已完成 |
| 首页可以访问 | 已完成 |
| 实验记录页面可以访问 | 已完成 |
| 任务问题、部署、验收清单页面可以跳转 | 已完成 |
| 自定义 CSS 能加载 | 已完成 |
| 执行 `hugo -D` 后能生成 `public/` | 已完成 |

## 实验截图

后续提交报告时，可以在这里补充以下截图：

- Hugo 本地运行成功截图
- 网站首页预览截图
- GitHub 仓库文件结构截图
- Git 提交记录截图
- 虚拟机或服务器部署访问截图

示例引用格式：

```markdown
![站点预览](/images/screenshow1.png)
![Git提交记录](/images/screenshot2.png)
```

## 实验小结

通过本次实验，完成了 Hugo 静态网站的本地运行和基础内容整理，了解了 `content/`、`static/`、`themes/`、`public/` 等目录的作用。实验中主要问题集中在环境变量、主题文件、样式加载和图片路径上。解决这些问题后，网站能够在本地正常预览，并可以继续交给部署同学上传到虚拟机或服务器中运行。
