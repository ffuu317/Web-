---
title: "部署与发布"
description: "把站点发布到可访问的地址（Nginx 或 GitHub Pages）。"
---

本页用于记录傅佳仪负责的虚拟机部署方案与操作步骤，重点对应 Linux 环境配置、Linux 账户配置、远程管理配置和 Web 站点部署。

## 对应验收项目

| 验收项目 | 本页记录重点 |
|---|---|
| Linux 环境配置 | Nginx 安装、系统环境、端口、防火墙 |
| Linux 账户配置 | 用户目录、站点目录、文件权限 |
| 远程管理配置 | SSH 登录、端口、上传和远程维护 |
| Web 站点部署 | 生成 `public/`、上传静态文件、配置 Nginx 并访问验证 |

## 方案 A：Linux + Nginx（虚拟机/云服务器）

**目标**：让站点可以通过外网访问（IP/域名 + 端口）。

建议步骤：

1. 安装 Nginx
2. 配置防火墙/安全组放行 80/443（或自定义端口）
3. 生成静态文件：`hugo -D`（输出到 `public/`）
4. 上传 `public/` 到服务器（例如 `/var/www/site/`）
5. Nginx 配置 `root` 指向站点目录，重载 Nginx

> 具体命令与配置片段请根据你们的服务器系统补充到这里（Ubuntu/CentOS 等）。

## Linux 环境配置（Nginx 安装）

部署站点前需要先在服务器上安装 Nginx，作为静态文件的 Web 服务器。

### 1. 更新软件源

安装前先更新 apt 软件包列表，确保能获取到最新版本的软件：

```bash
sudo apt update
```

更新过程中会从 Ubuntu 官方源下载各个组件的包信息，完成后显示有多少个包可以升级。

![apt update 更新软件源](/images/screenshot-nginx-apt-update.svg)
<p class="img-caption">图 1　sudo apt update 更新完成，共 111 个包可升级</p>

### 2. 安装 Nginx

软件源更新完成后，安装 Nginx：

```bash
sudo apt install nginx -y
```

安装完成后，用 `nginx -v` 验证是否安装成功并查看版本号。

![安装 Nginx](/images/screenshot-nginx-install.svg)
<p class="img-caption">图 2　Nginx 1.24.0 安装成功</p>

### 3. 验证 Nginx 服务

安装完成后，Nginx 服务会自动启动。可以用以下命令确认：

```bash
# 查看 Nginx 服务状态
sudo systemctl status nginx

# 访问本地 80 端口，测试默认页面是否正常
curl http://localhost/
```

## Linux 账户配置

确认当前登录用户和 SSH 环境，为后续部署做准备。

### 1. 查看当前用户

```bash
# 查看当前登录的用户名
whoami

# 查看 /home 目录下有哪些用户
ls /home
```

![账户与 SSH 环境检查](/images/screenshot-account-check.svg)
<p class="img-caption">图 3　查看当前用户、用户目录和 sshd 路径</p>

### 2. 确认 SSH 服务存在

```bash
# 检查 sshd 可执行文件路径
which sshd
```

如果输出 `/usr/sbin/sshd`，说明 SSH 服务端已经安装。

### 3. 遇到的难点与解决过程

**难点：命令输错导致 command not found**

刚开始输入 `Is /home`（把小写的 L 写成了大写的 i），终端提示 `Is: command not found`。

```bash
# 错误写法（大写 i 和小写 L 搞混了）
Is /home
# Is: command not found
```

**解决方法**：Linux 命令都是小写的，列表命令是 `ls`（list 的缩写，字母 L 的小写），不是 `Is`。改成正确的小写 `ls /home` 就能正常显示用户目录了。

> 💡 小提示：在终端里，`l`（L 的小写）和 `I`（i 的大写）视觉上很像，输命令时注意区分。不确定命令时可以按 `Tab` 键让终端自动补全。

## 远程管理配置（SSH）

为了方便远程登录服务器和上传站点文件，需要配置好 SSH 服务。以下是本次实验中的 SSH 安全配置要点。

### 1. 备份与编辑配置文件

修改前先备份原始配置，改错了可以随时恢复：

```bash
# 备份原配置（防止改错）
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

**编辑方式**：如果 `nano` 无法修改，可以改用 VS Code 终端直接编辑：

```bash
# 用 VS Code 终端编辑（推荐）
code /etc/ssh/sshd_config

# 或用 vi/vim 编辑
sudo vi /etc/ssh/sshd_config
```

> **vi/vim 操作提示**：按 `i` 进入编辑模式；修改后按 `Esc` 退出编辑，输入 `:wq` 保存退出。

### 2. 修改 SSH 端口

把默认的 22 端口改为自定义端口（例如 2222），降低被暴力扫描的概率：

```text
Port 2222
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

![SSH 端口配置](/images/screenshot-ssh-port.svg)
<p class="img-caption">图 4　sshd_config 中 Port 改为 2222</p>

### 3. 禁止 root 直接登录

出于安全考虑，关闭 root 账号的 SSH 登录权限，改为用普通用户登录后再 `sudo` 提权：

```text
PermitRootLogin no
```

![禁止 root 登录](/images/screenshot-ssh-root.svg)
<p class="img-caption">图 5　PermitRootLogin 设置为 no</p>

### 4. 开启密码认证

如果暂时还没有配置密钥登录，可以保留密码认证，确保能正常登录：

```text
PasswordAuthentication yes
```

![密码认证开启](/images/screenshot-ssh-password.svg)
<p class="img-caption">图 6　PasswordAuthentication 设置为 yes</p>

### 5. 重启 SSH 服务使配置生效

修改完 `sshd_config` 后，需要重启 sshd 服务才能让新配置生效：

```bash
# 检查配置是否有语法错误
sudo sshd -t

# 重启 sshd 服务
sudo service ssh restart
```

### 6. 检查 SSH 服务状态与端口监听

修改端口后，验证服务是否正确运行并监听新端口：

```bash
# 检查 SSH 服务状态
sudo service ssh status

# 检查端口监听情况（确认 2222 是否被监听）
sudo ss -tlnp | grep 2222

# 查看 sshd_config 中的端口配置
sudo grep -E "^Port|^#Port" /etc/ssh/sshd_config
```

![SSH 服务状态与端口检查](/images/screenshot-ssh-status.svg)
<p class="img-caption">图 7　检查 SSH 服务状态，发现仍监听端口 22（因 ssh.socket 干扰）</p>

### 7. 禁用 ssh.socket 触发器

如果发现修改了端口但服务仍监听旧端口，可能是 `ssh.socket` 在干扰。需要禁用 socket 模式：

```bash
# 停止并禁用 ssh.socket
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket

# 重启 SSH 服务
sudo service ssh restart

# 再次检查端口监听
sudo ss -tlnp | grep 2222
```

![禁用 ssh.socket](/images/screenshot-ssh-disable-socket.svg)
<p class="img-caption">图 8　禁用 ssh.socket 触发器后，端口配置才能正确生效</p>

### 8. 本地登录验证

在服务器上用新端口测试 SSH 登录，确认配置生效：

```bash
# 使用新端口登录本地
ssh localhost -p 2222
```

首次登录会提示确认主机密钥，输入 `yes` 确认即可。

![SSH 端口登录成功](/images/screenshot-ssh-login.svg)
<p class="img-caption">图 9　成功通过端口 2222 登录到 Ubuntu 24.04 LTS</p>

### 9. 配置 Linux 环境变量

在服务器上配置个人环境变量，方便后续开发：

```bash
# 添加环境变量到 bashrc
echo 'export MY_NAME="ffuu"' >> ~/.bashrc

# 立即生效（不需要重启终端）
source ~/.bashrc

# 验证环境变量是否生效
echo $MY_NAME
```

![配置环境变量](/images/screenshot-ssh-env.svg)
<p class="img-caption">图 10　配置环境变量 MY_NAME 并验证成功</p>

### 10. 防火墙放行新端口

改了端口后，记得在防火墙中放行新的 SSH 端口，否则会连不上：

```bash
# firewalld 方式（CentOS / openEuler 等）
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --reload

# ufw 方式（Ubuntu 等）
sudo ufw allow 2222/tcp
sudo ufw reload
```

### 11. 登录验证

配置完成后，在本地用新端口测试登录：

```bash
ssh -p 2222 用户名@服务器IP
```

> ⚠️ 重要提醒：修改 SSH 端口和 `PermitRootLogin` 之前，**先不要关闭当前已登录的终端**，用另一个终端测试登录，确认能正常连上再退出，避免把自己锁在服务器外面。

## 傅佳仪部署任务记录

| 任务 | 记录内容 |
|---|---|
| Linux 环境 | 虚拟机系统版本、Nginx 安装方式、防火墙/端口配置 |
| Linux 账户 | 部署用户、站点目录、文件读写权限 |
| 远程管理 | SSH 登录命令、端口、文件上传方式 |
| 站点部署 | `public/` 生成、上传路径、Nginx `root`、访问地址 |

## 部署中遇到的难点与解决办法

下面是本次实验中实际遇到的问题和对应的解决过程，方便后续部署参考。

### 1. 命令输错导致 command not found

| 项目 | 内容 |
|---|---|
| 问题现象 | 输入 `Is /home`，终端提示 `Is: command not found` |
| 原因分析 | 把小写字母 `l`（L）和大写字母 `I`（i）搞混了，Linux 命令都是小写的 |
| 解决方法 | 改成正确的 `ls /home`（list 的缩写，L 的小写），正常显示用户目录 |
| 经验总结 | 终端字体下 `l` 和 `I` 视觉上很像，不确定时按 `Tab` 键让终端自动补全 |

### 2. SSH 端口修改不生效（被 ssh.socket 覆盖）

| 项目 | 内容 |
|---|---|
| 问题现象 | 修改了 `sshd_config` 的 `Port 2222`，但 `ss -tlnp` 显示仍监听端口 22 |
| 原因分析 | Ubuntu 默认启用了 `ssh.socket`（socket 激活模式），修改 `sshd_config` 不会影响 socket 的监听端口 |
| 解决方法 | `sudo systemctl stop ssh.socket` + `sudo systemctl disable ssh.socket`，然后 `sudo service ssh restart` |
| 经验总结 | 改端口前先确认系统用的是 `ssh.service` 还是 `ssh.socket`，两者是两套机制 |

### 3. nano 编辑器无法修改配置文件

| 项目 | 内容 |
|---|---|
| 问题现象 | `nano` 无法正常编辑 `/etc/ssh/sshd_config` |
| 解决方法 | 改用 VS Code 终端：`code /etc/ssh/sshd_config`；或用 `vi/vim` 编辑 |
| vi 操作提示 | 按 `i` 进入编辑模式，修改后按 `Esc` 退出编辑，输入 `:wq` 保存退出 |

### 4. 其他常见问题速查

| 模块 | 常见难点 | 解决办法 |
|---|---|---|
| Linux 环境配置 | 安装 Nginx 后访问不到默认页 | 检查 Nginx 是否启动：`systemctl status nginx`；检查防火墙和安全组是否放行端口 |
| Linux 环境配置 | 服务器没有 Hugo 命令 | 可在本地执行 `hugo -D` 生成 `public/`，只把静态文件上传到服务器 |
| Linux 账户配置 | 普通用户无法写入 `/var/www/` | 使用管理员权限创建站点目录，或把目录属主改为当前部署用户 |
| Linux 账户配置 | Nginx 读取站点文件失败 | 检查目录权限，确保 Nginx 运行用户对站点目录有读权限 |
| 远程管理配置 | SSH 登录失败 | 核对 IP、端口、用户名和密码/密钥；确认服务器 SSH 服务已开启 |
| 远程管理配置 | 文件上传后目录不对 | 上传前确认目标路径，上传后用 `ls` 检查 `index.html` 是否在 Nginx root 目录下 |
| Web 站点部署 | 首页 404 或样式丢失 | 检查 `root` 是否指向包含 `index.html` 的目录，检查 `/css/custom.css` 是否可访问 |
| Web 站点部署 | 修改后页面仍是旧版 | 重新执行 `hugo -D`，上传新的 `public/`，并清理浏览器缓存 |

## 方案 B：GitHub Pages（可选）

如果你们希望免服务器部署，可以考虑 GitHub Pages。常见做法：

- 让 GitHub Pages 直接从仓库根目录或 `public/` 发布
- 或用 CI（GitHub Actions）自动构建并发布

后续如果你把 Pages 的发布方式定下来，我可以把配置补齐并帮你一键跑通。
