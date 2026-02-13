前言

说起密码管理工具，可能很多人第一时间想到的是浏览器自带的密码保存功能。但它在不同设备间的同步和安全性上，总有些让人不放心。对我而言，真正开始认真管理密码，是从苹果的 iCloud 钥匙串开始的。它的生态集成做得很好，但缺点也同样明显：一旦你走出苹果的围墙，比如同时使用 Windows 电脑和安卓手机，它就无法跨平台工作。

随着技术发展，一种新的身份验证方式——通行密钥（Passkeys） 正悄然成为主流。就像其官网所说，它让我们可以用指纹、人脸识别或设备PIN码来登录，比传统密码更安全、更方便。

越来越多的网站和应用开始支持 Passkeys。对于苹果全家桶用户来说，体验是无缝的，因为系统本身就集成了。但对像我这样跨平台（IOS + Android + Windows）的用户，问题就来了：如何让手机上的 Passkeys 能顺利地在电脑上使用？

于是，寻找一个能 自托管、支持 Passkeys、并且能跨平台使用 的密码管理方案，就成了我的当务之急。经过一番研究，Vaultwarden 进入了我的视野。

为什么选择 Vaultwarden？

Vaultwarden 是一个用 Rust 语言编写的开源项目，它是官方 Bitwarden 服务器的一个轻量级替代品。它的优势非常突出：

· 资源占用极低：即使运行在树莓派或玩客云这样的低性能设备上也能流畅运行。
· 功能完整：几乎实现了 Bitwarden 的所有核心功能，包括对 Passkeys 的完美支持。
· 数据自主掌控：所有密码数据都存储在你的私人服务器上，不再担心第三方服务的安全或政策变动。

下面，我就以最快捷的方式，带你在自己的服务器上部署 Vaultwarden。前提是你已经安装好了 Docker 和 Caddy（或其他支持自动 HTTPS 的反向代理工具）。

环境准备

· 操作系统：推荐 Linux（如 Ubuntu、Debian、CentOS 等），当然 Windows 和 macOS 也可以，但 Linux 服务器是最佳选择。
· 硬件要求：Vaultwarden 非常轻量，1GB 内存和 1GHz 的 CPU 就足以保证流畅运行。硬盘空间建议至少 10GB，具体取决于你的密码和附件数量。
· 网络配置：确保服务器的 80 和 443 端口（或其他你指定的端口）可以从外部访问。

部署 Vaultwarden（Docker 方式）

使用 Docker 部署 Vaultwarden 是最简单、最推荐的方式。它可以将应用与系统环境隔离，方便管理和升级。

1. 安装 Docker

如果你的系统尚未安装 Docker，可以参考 官方文档 或网上众多的教程进行安装。

2. 使用 docker run 快速启动（简单直接）

这是最快捷的启动方式，适合快速测试。在终端执行以下命令：

```bash
docker run -d \
  --name vaultwarden \
  -v /path/to/your/data/:/data/ \
  -p 8080:80 \
  vaultwarden/server:latest
```

命令解释：

· -d：后台运行容器。
· --name vaultwarden：为容器命名为 vaultwarden。
· -v /path/to/your/data/:/data/：请务必将 /path/to/your/data/ 替换为你服务器上的一个实际路径，用于持久化存储密码数据库等重要数据。
· -p 8080:80：将容器的 80 端口映射到你服务器的 8080 端口。之后你就可以通过 http://你的服务器IP:8080 来访问 Vaultwarden 了。

3. 使用 docker-compose 部署（推荐，更便于管理）

如果你希望更规范地管理配置，推荐使用 docker-compose。首先，在你喜欢的位置创建一个目录（如 vaultwarden），并在该目录下创建 docker-compose.yml 文件，内容如下：

```yaml
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      TZ: 'Asia/Shanghai' # 设置时区
      ADMIN_TOKEN: 'your_strong_admin_token_here' # 管理员面板的访问密码，必须设置！
      SIGNUPS_ALLOWED: 'true' # 是否允许新用户注册，建议注册完自己的账号后改为 'false'
      # SIGNUPS_VERIFY: 'true' # 是否验证邮箱，需先配置SMTP，否则留空或注释掉
      # DOMAIN: 'https://your-domain.com' # 你的最终访问域名，有域名时建议设置
    volumes:
      - './data:/data' # 将数据挂载到当前目录下的 data 文件夹
    ports:
      - '127.0.0.1:8888:80' # 只监听本地端口，供反向代理使用，更安全
```

关键配置说明：

· ADMIN_TOKEN：这是你的管理员面板密码，非常重要！ 你可以使用官方推荐的方式生成一个加密后的密码，以提高安全性：
  ```bash
  docker run --rm -it vaultwarden/server /vaultwarden hash
  ```
  按照提示输入你的管理员密码，它会返回一串以 $argon2 开头的加密字符串，将这整串字符串复制到 ADMIN_TOKEN 的值即可。
· ports: '127.0.0.1:8888:80'：我们将 Vaultwarden 的 80 端口映射到了宿主机的 127.0.0.1:8888。这意味着只能通过本机访问这个端口，外网无法直接连接，必须通过后面的反向代理。这是一种安全实践。
· 关于 WebSocket：Vaultwarden 默认已支持 WebSocket，无需额外配置。

在 docker-compose.yml 文件所在目录下，执行以下命令启动服务：

```bash
docker-compose up -d
```

配置反向代理（以 Caddy 为例）

为了通过域名安全地访问 Vaultwarden（启用 HTTPS），我们需要配置反向代理。Caddy 以其自动申请和续签 SSL 证书的能力而闻名，配置非常简单。

假设你的域名是 vault.your-domain.com，并且 DNS 已经正确解析到你的服务器 IP。在你的 Caddyfile 中添加如下配置：

```
vault.your-domain.com {
    # 如果你希望从 Let's Encrypt 自动获取 SSL 证书，取消下面这行的注释，并替换为你的邮箱
    # tls your-email@example.com

    # 开启压缩
    encode gzip

    # 将所有请求反向代理到本地的 Vaultwarden 服务
    reverse_proxy 127.0.0.1:8888 {
        # 传递真实 IP，对日志和 fail2ban 等安全工具至关重要
        header_up X-Real-IP {remote_host}
    }
}
```

如果你不希望管理员面板（/admin）暴露在公网，可以添加以下规则来屏蔽它，只允许内网访问：

```
vault.your-domain.com {
    encode gzip

    # 处理非 /admin 的路径，正常代理
    @notAdmin not path /admin*
    handle @notAdmin {
        reverse_proxy 127.0.0.1:8888 {
            header_up X-Real-IP {remote_host}
        }
    }

    # 对 /admin 路径直接返回 404，从而屏蔽外网访问
    @admin path /admin*
    handle @admin {
        respond "Not Found" 404
    }
}
```

保存 Caddyfile 并重载 Caddy 配置，你的 Vaultwarden 就应该可以通过你的域名安全地访问了。

安全与维护

1. 启用 HTTPS：通过上面的 Caddy 配置，我们已经强制启用了 HTTPS，这是最基本也是最重要的一步。
2. 关闭注册：在你自己注册完主账号后，务必将 docker-compose.yml 中的 SIGNUPS_ALLOWED 改为 false，然后重启容器，防止他人随意注册。
3. 定期更新：时不时拉取最新的 Vaultwarden 镜像并重启容器，以获取最新的功能和安全更新。
   ```bash
   docker-compose pull
   docker-compose up -d
   ```
4. 备份数据：./data 目录下存放着你的所有密码数据，请务必定期备份这个目录。

结语

至此，一个属于你自己的、安全、跨平台且支持最新通行密钥（Passkeys）的密码管理系统就搭建完成了。通过 Vaultwarden，你不仅摆脱了对第三方服务的依赖，也获得了对个人数字身份的最高掌控权。

你可以在任何设备上安装 Bitwarden 的官方客户端（浏览器扩展、桌面应用、手机 App），然后在登录界面选择“自托管”，输入你自己的域名，就能愉快地使用了！