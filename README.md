# 微信读书挑战会员助手

只为便宜一点买微信读书会员。

> 这是 [jqknono/weread-challenge-selenium](https://github.com/jqknono/weread-challenge-selenium)
> 的个人分支：**只构建并发布 `linux/arm64`**（树莓派等 ARM 开发板），镜像发布在
> `lonemoonspace/weread-challenge`。需要 `amd64` 请使用上游仓库。

![只为便宜一点买微信读书会员](https://img1.techfetch.dev/blog/202412261741639.gif)

## 快速开始

```bash
# 创建工作目录
mkdir -p $HOME/weread-challenge && cd $HOME/weread-challenge

# 下载配置文件
wget https://raw.githubusercontent.com/lonemoonspace/weread-challenge-selenium/main/docker-compose.yml

# 启动服务
docker compose up -d

# 创建定时任务
(crontab -l 2>/dev/null; echo "00 */6 * * *  cd $HOME/weread-challenge && docker compose up -d") | crontab -

# 扫描$HOME/weread-challenge/.weread下生成的登录二维码, 开始自动阅读
```

## 微信读书规则

- 离线阅读计入总时长，但需要联网上报
- 网页版、墨水屏、小程序、听书、有声书收听**都计入总时长**
- 对单次自动阅读或收听时长过长的行为，平台将结合用户行为特征判断，**过长部分不计入总时长**
- 当日阅读超过 **5 分钟** 才算作有效阅读天数
- 付费 5 元立即获得 2 天会员，后续 30 日内打卡 29 天，读书时长超过 30 小时，可获得 30 天会员和 30 书币
- 付费 50 元立即获得 30 天会员，后续 365 日内打卡 360 天，读书时长超过 300 小时，可获得 365 天会员和 500 书币

根据实际操作，还有如下未明确说明的特点：

- **打卡周期**：第 29 日打卡后立即获得读书会员奖励，并可立即开始下一轮挑战会员打卡，无需等待第 31 日开始下一轮挑战，第 29 日的打卡既算上一轮的打卡，也算下一轮的打卡
- **年度计算**：除第一轮需 29 日外，后续每 28 日即可获得 32 日会员，1+28×13=365，一年可完成 13 轮，花费 65 元，获得 32×13=416 天会员和 390 书币
- **性价比**：更划算的仍然是年卡挑战会员，但周期更长，风险更大

## 工具特性

- 使用有头浏览器
- 支持本地浏览器和远程浏览器
- 随机浏览器宽度和高度
- 支持等待登录
- 支持登录二维码刷新
- 支持保存 cookies
- 支持加载 cookies
- 支持选择最近阅读的第 X 本书开始阅读
- 默认选择书架第 2 本书开始阅读
- 支持随机选择书籍开始阅读
- 支持自定义默认兜底阅读链接
- 支持自动阅读
- 支持跳到下一章
- 支持读完跳回第一章继续阅读
- 支持选择阅读速度
- 随机单页阅读时间
- 随机翻页时间
- 每分钟截图当前界面
- 支持日志
- 支持定时任务
- 支持设置阅读时间
- 支持邮件通知
- 支持 Bark 推送通知（iOS）
- 支持 ntfy 推送通知（Android / iOS / 桌面）
- 多平台支持: `linux | windows | macos`
- 支持架构: `arm64`
- 支持浏览器: `chrome | MicrosoftEdge | firefox | safari`
- 支持多用户
- 异常时强制刷新
- 可选的使用统计上报（默认关闭）

## CLI 子命令

| 子命令 | 作用 | 关键参数 |
| --- | --- | --- |
| `run` | 执行微信读书自动阅读主流程 | 沿用现有环境变量 |
| `schedule` | 生成计划任务命令，支持 `windows` / `macos` / `linux` | `--name` `--every` `--workdir` `--weread-duration` |
| `help` / `-h` / `--help` | 显示帮助 | 可跟 `run` / `schedule` |

> npm 上的 `weread-selenium-cli` 由上游作者发布，本分支不发布 npm 包。`npx` / `npm install -g`
> 安装的是上游版本；想运行本分支代码，请克隆本仓库后执行 `npm install && npm link`，或使用 Docker 镜像。

```bash
# 查看总帮助
npx weread-selenium-cli -h

# 运行主流程
npx weread-selenium-cli run

# run 参数会覆盖同名环境变量
npx weread-selenium-cli run --weread-browser firefox --weread-duration 30

# 生成 Windows 计划任务创建命令
npx weread-selenium-cli schedule --name weread-hourly --every 60 --platform windows

# 计划任务仅支持附加阅读时长参数
npx weread-selenium-cli schedule --name weread-hourly --every 240 --platform windows --weread-duration 10
```

`schedule` 需要在目标平台上运行；例如 Windows 计划任务要在 Windows 上执行该子命令。
`schedule` 只生成创建/验证/删除命令，不会直接注册系统计划任务。
`schedule` 仅支持把 `--weread-duration` 追加到生成出来的 `weread-selenium-cli run` 命令中，不支持其他 `run` 参数。
Windows 默认会生成一条 `schtasks` 创建命令：先按当天某个时间点开始，再按指定间隔重复执行，重复持续时间固定为 `8760:00`。
`schedule` 未显式传 `--workdir` 时，默认使用当前用户 `HOME` 作为工作目录。
如果 Windows 创建命令执行时报 `Access is denied`，请改在“以管理员身份运行”的终端里执行生成出来的创建命令。
`run` 支持把现有环境变量改写成参数形式，优先级为：CLI 参数 > 环境变量 > 默认值。参数既支持 `--weread-browser` 这种 kebab-case，也支持 `--WEREAD_BROWSER` 这种原始环境变量名。
`weread-selenium-cli` 是主命令，旧全局命令 `weread-challenge` 继续保留，等价于执行 `weread-selenium-cli run`。
本地运行在未设置 `WEREAD_DATA_DIR` 时，会优先使用当前工作目录下已存在的 `.weread`，其次复用旧版 `data/`，否则新建 `.weread`。

## Linux

### 直接运行

```bash
# 安装 Node.js 和 npm
sudo apt update
sudo apt install nodejs npm

# 创建运行文件夹
mkdir -p $HOME/Documents/weread-challenge
cd $HOME/Documents/weread-challenge

# 安装 npm 包
npm install -g weread-selenium-cli

# 设置环境变量并运行
export WEREAD_BROWSER="chrome"
weread-selenium-cli run
```

也可不全局安装，直接使用 npx 运行：

```bash
npx weread-selenium-cli run
```

已存在的旧全局命令 `weread-challenge` 仍可直接运行。

如果你直接运行当前仓库源码，可使用：

```bash
# 安装依赖
npm install

# 本地浏览器运行
npm run dev

# 本地浏览器 + DEBUG 日志
npm run debug

# 使用 package.json 内置示例参数运行
# 注意：npm run start 中的 WEREAD_REMOTE_BROWSER 是示例地址，需按实际环境调整
npm run start
```

### Docker Compose 运行

```yaml
services:
  app:
    image: docker.io/lonemoonspace/weread-challenge:latest
    pull_policy: always
    environment:
      - WEREAD_REMOTE_BROWSER=http://selenium:4444
      - WEREAD_DURATION=68
    volumes:
      - ./.weread:/app/.weread
    depends_on:
      selenium:
        condition: service_healthy
    dns:
      - 223.5.5.5

  selenium:
    image: selenium/standalone-chromium:latest
    pull_policy: if_not_present
    shm_size: 2gb
    environment:
      - SE_ENABLE_TRACING=false
      - SE_BIND_HOST=false
      - SE_JAVA_DISABLE_HOSTNAME_VERIFICATION=false
    dns:
      - 223.5.5.5
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4444/status"]
      interval: 5s
      timeout: 60s
      retries: 10
```

将以上内容保存为 `docker-compose.yml` 文件，然后运行：

```bash
docker compose up -d
```

首次启动后，需要微信扫描二维码登录，二维码保存在 `./.weread/login.png`
如果你仍在沿用旧版 `./data:/app/data` 挂载，且没有显式设置 `WEREAD_DATA_DIR`，当前版本会继续复用 `/app/data`；新部署示例统一使用 `./.weread:/app/.weread`。

### Docker 运行

```bash
# 创建网络
docker network create weread-challenge-net

# 启动 Selenium 服务
docker run --restart always -d --name selenium-live \
  --shm-size="2g" \
  --network weread-challenge-net \
  --hostname selenium-live \
  --dns 119.29.29.29 \
  --dns 223.5.5.5 \
  -p 4444:4444 \
  -p 7900:7900 \
  -e SE_ENABLE_TRACING=false \
  -e SE_BIND_HOST=false \
  -e SE_JAVA_DISABLE_HOSTNAME_VERIFICATION=false \
  -e SE_NODE_MAX_INSTANCES=10 \
  -e SE_NODE_MAX_SESSIONS=10 \
  -e SE_NODE_OVERRIDE_MAX_SESSIONS=true \
  selenium/standalone-chromium:latest

# 运行微信读书挑战
docker run --rm --name user-read \
  -v $HOME/weread-challenge/user/.weread:/app/.weread \
  -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 \
  -e WEREAD_DURATION=68 \
  docker.io/lonemoonspace/weread-challenge:latest

# 添加第二个用户
docker run --rm --name user2-read \
  -v $HOME/weread-challenge/user2/.weread:/app/.weread \
  -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 \
  -e WEREAD_DURATION=68 \
  docker.io/lonemoonspace/weread-challenge:latest
```

首次启动后，需要微信扫描二维码登录，二维码保存在 `./.weread/login.png`

### 创建定时任务

有两种方式添加定时任务：

- **docker-compose 方式**：通过 `docker-compose` 启动，浏览器环境配置简单，一键启动，每个浏览器实例对应一个用户
- **docker 方式**：通过 `docker` 启动，工具使用远程浏览器特性连接到浏览器实例，浏览器环境可能需要自行调整，支持单浏览器多用户

#### docker-compose 方式

```bash
WORKDIR=$HOME/weread-challenge
mkdir -p $WORKDIR
cd $WORKDIR
cat > $WORKDIR/docker-compose.yml <<EOF
services:
  app:
    image: docker.io/lonemoonspace/weread-challenge:latest
    pull_policy: always
    environment:
      - WEREAD_REMOTE_BROWSER=http://selenium:4444
      - WEREAD_DURATION=68
    volumes:
      - ./.weread:/app/.weread
    depends_on:
      selenium:
        condition: service_healthy

  selenium:
    image: selenium/standalone-chromium:latest
    pull_policy: if_not_present
    shm_size: 2gb
    environment:
      - SE_ENABLE_TRACING=false
      - SE_BIND_HOST=false
      - SE_JAVA_DISABLE_HOSTNAME_VERIFICATION=false
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4444/wd/hub/status"]
      interval: 5s
      timeout: 60s
      retries: 10
EOF
# 首次启动后, 需微信扫描二维码登录, 二维码保存在 $HOME/weread-challenge/.weread/login.png
# 每隔6个小时, 阅读68分钟
(crontab -l 2>/dev/null; echo "00 */6 * * *  cd $WORKDIR && docker compose up -d") | crontab -
```

#### docker 方式

```bash
# 创建网络
docker network create weread-challenge-net

# 启动浏览器
docker run --restart always -d --name selenium-live \
  --network weread-challenge-net \
  --hostname selenium-live \
  --shm-size="2g" \
  -p 4444:4444 \
  -p 7900:7900 \
  -e SE_ENABLE_TRACING=false \
  -e SE_BIND_HOST=false \
  -e SE_JAVA_DISABLE_HOSTNAME_VERIFICATION=false \
  -e SE_NODE_MAX_INSTANCES=3 \
  -e SE_NODE_MAX_SESSIONS=3 \
  -e SE_NODE_OVERRIDE_MAX_SESSIONS=true \
  -e SE_SESSION_REQUEST_TIMEOUT=10 \
  -e SE_SESSION_RETRY_INTERVAL=3 \
  selenium/standalone-chromium:latest

WEREAD_USER="user"
mkdir -p $HOME/weread-challenge/$WEREAD_USER/.weread
# 首次启动后, 需微信扫描二维码登录, 二维码保存在 $HOME/weread-challenge/$WEREAD_USER/.weread/login.png
# 每隔6个小时, 阅读68分钟
(crontab -l 2>/dev/null; echo "00 */6 * * * docker run --rm --name ${WEREAD_USER}-read -v $HOME/weread-challenge/${WEREAD_USER}/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=68 -e WEREAD_USER=${WEREAD_USER} docker.io/lonemoonspace/weread-challenge:latest") | crontab -
```

crontab 示例：

```bash
00 01 * * * docker run --rm --name user1-read -v /home/test/weread-challenge/user1/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=180 -e WEREAD_USER=user1 -e WEREAD_SELECTION=2 -e ENABLE_EMAIL=true -e EMAIL_SMTP=smtp.mail.me.com -e EMAIL_USER=user1@icloud.com -e EMAIL_PASS=aaaa-bbbb-cccc-dddd -e EMAIL_PORT=587 -e EMAIL_TO=you@example.com docker.io/lonemoonspace/weread-challenge:latest

00 01 * * * docker run --rm --name user2-read -v /home/test/weread-challenge/user2/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=180 -e WEREAD_USER=user2 -e WEREAD_SELECTION=2 -e ENABLE_EMAIL=true -e EMAIL_SMTP=smtp.mail.me.com -e EMAIL_USER=user2@icloud.com -e EMAIL_PASS=aaaa-bbbb-cccc-dddd -e EMAIL_PORT=587 -e EMAIL_TO=you@example.com docker.io/lonemoonspace/weread-challenge:latest

00 01 * * * docker run --rm --name user1-read -v /home/test/weread-challenge/user1/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=180 -e WEREAD_USER=user1 -e WEREAD_SELECTION=2 -e BARK_KEY=your-bark-key-here docker.io/lonemoonspace/weread-challenge:latest
```

## Windows

```powershell
# 安装 Node.js
winget install -e --id OpenJS.NodeJS.LTS

# 创建运行文件夹
New-Item -ItemType Directory -Force -Path "$HOME\Documents\weread-challenge"
Set-Location "$HOME\Documents\weread-challenge"

# 安装 npm 包
npm install -g weread-selenium-cli

# 设置环境变量并运行
$env:WEREAD_BROWSER="MicrosoftEdge"
weread-selenium-cli run
```

Docker 运行方式与 Linux 相同。

## MacOS

```bash
# 安装 Node.js
brew install node

# 创建运行文件夹
mkdir -p $HOME/Documents/weread-challenge
cd $HOME/Documents/weread-challenge

# 安装 npm 包
npm install -g weread-selenium-cli

# 启用 Safari 自动化支持
sudo safaridriver --enable

# 设置环境变量并运行
export WEREAD_BROWSER="safari"
weread-selenium-cli run
```

Docker 运行同 Linux.

## 推送通知

脚本支持两种推送渠道，可以只用一个，也可以同时开启：

| 渠道 | 适用平台 | 启用方式 |
| --- | --- | --- |
| ntfy | Android / iOS / 桌面（**Android 推荐**） | 设置 `NTFY_TOPIC` |
| Bark | iOS | 设置 `BARK_KEY` |

推送统一走通知层：配置了几个渠道就并发推送到几个渠道，单个渠道失败不会影响其他渠道，也不会中断阅读流程。

### ntfy 推送（Android 推荐）

[ntfy](https://ntfy.sh) 是开源的推送服务，Android 端有官方 App，无需注册即可使用公共服务器 `https://ntfy.sh`，也可以自建服务。

1. 在 Android 手机安装 ntfy App（F-Droid / Google Play 搜索 `ntfy`）
2. 取一个不易被猜到的主题名（topic 等价于密码，只允许字母、数字、`_`、`-`，长度不超过 64），例如 `weread-yourname-a8f3k`
3. 在 App 中点击 `+` 订阅该主题
4. 设置环境变量 `NTFY_TOPIC` 为该主题名

**简化配置**：只需设置 `NTFY_TOPIC` 即可启用 ntfy 推送，无需额外启用开关。

```bash
export NTFY_TOPIC="weread-yourname-a8f3k"
npx weread-selenium-cli run
```

可选配置：

- `NTFY_SERVER`：自建 ntfy 服务地址，默认 `https://ntfy.sh`
- `NTFY_TOKEN`：ntfy Access Token；或改用 `NTFY_USERNAME` + `NTFY_PASSWORD` 走 Basic Auth
- `NTFY_PRIORITY`：覆盖默认优先级，取值 `1-5` 或 `min|low|default|high|urgent`
- `NTFY_TAGS`：覆盖默认标签，逗号分隔，例如 `book,heavy_check_mark`

默认映射关系：

| 事件 | Bark level | ntfy 优先级 | 默认 ntfy 标签 |
| --- | --- | --- | --- |
| 脚本启动 / 登录成功 / 阅读完成 | `active` | `3`（default） | `rocket` / `bell` / `tada` |
| 登录失败 / 运行报错 | `critical` | `5`（urgent） | `rotating_light` |
| 检测到登录二维码 | `active` | `3`（default） | `bell` |

> ntfy 的 JSON 发布接口只接受 1-5 的数字优先级，本项目在发送前会把 `min/low/default/high/urgent/max` 别名与 `NTFY_PRIORITY` 配置统一归一化为数字，避免旧版本中字符串别名导致的 `400 {"code":40024}` 错误。

登录二维码推送时，ntfy 通知会带上「打开登录链接」和「查看二维码」两个按钮，点击通知本身也会直接打开登录链接。

#### 使用自建 ntfy 服务器

自建服务器只需要把地址填进 `NTFY_SERVER`（**只填站点根地址，不要带主题名，也不要带末尾斜杠**）：

```bash
export NTFY_SERVER="https://ntfy.example.com"
export NTFY_TOPIC="weread-yourname-a8f3k"
npx weread-selenium-cli run
```

前提条件与常见坑：

- 自建服务必须能通过 HTTPS 访问，且证书有效；自签证书会因 Node 默认校验证书而报 `unable to verify the issuer`，请用 Let's Encrypt 等受信任证书
- 若反向代理把 ntfy 挂在子路径（如 `https://example.com/ntfy`），`NTFY_SERVER` 需带上该子路径
- 若服务端开启了鉴权（`auth-default-access` 为 `deny-all` 等），需要同时配置 `NTFY_TOKEN`，或 `NTFY_USERNAME` + `NTFY_PASSWORD`
  - 典型症状：发布时返回 `403 {"code":40301,"error":"forbidden"}`，就是 ACL 拒绝了匿名访问，必须补 Token
  - `/v1/config` 里的 `enable_login` / `require_login` **不代表**匿名能否发布，判断权限要看 `/v1/account` 或直接试发一条
- 服务端建议设置 `base-url: "https://你的域名"`，并在反向代理/Cloudflare 后面开启 `behind-proxy: true`，否则所有访客会被当成同一个 IP 计入限流
- 主题名不能与 ntfy 保留路径重名（`docs`、`static`、`file`、`app`、`metrics`、`account`、`settings`、`signup`、`login`、`v1`）
- 未开启鉴权的服务器对公网开放：主题名等价于密码，务必使用随机后缀，避免被他人猜中订阅或投递
- Android App 需先添加自建服务器：设置 → `Manage users` → 添加服务器地址并登录（若需要），再订阅 `NTFY_TOPIC` 对应的主题

连通性自查：

```bash
# 1. 服务端健康检查
curl -s https://ntfy.example.com/v1/health

# 2. 用 curl 直接投递一条测试消息，手机上应立即收到
curl -d "测试消息" https://ntfy.example.com/weread-yourname-a8f3k

# 3. 用本项目真实的推送链路测试（不启动浏览器）
NTFY_SERVER=https://ntfy.example.com NTFY_TOPIC=weread-yourname-a8f3k \
  node -e "require('./src/weread-challenge.js').notify('微信读书挑战','ntfy 连通性测试',{subtitle:'连通测试',sound:'success'})"
```

### Bark 推送（iOS）

Bark 是一个 iOS 设备上的推送服务，可以通过简单的 HTTP 请求向 iPhone 发送通知。本工具支持通过 Bark 推送运行状态和结果。

#### 配置 Bark

1. 在 iPhone 上下载并安装 Bark App
2. 打开 Bark App，获取推送密钥（通常是设备码）
3. 设置环境变量 `BARK_KEY` 为获取的密钥
4. 可选：设置 `BARK_SERVER` 为自定义 Bark 服务器地址（默认使用官方服务器 `https://api.day.app`）

**简化配置**：只需设置 `BARK_KEY` 即可启用 Bark 推送，无需额外启用开关。

### 使用示例

#### 直接运行（Linux/MacOS/Windows）

```bash
export NTFY_TOPIC="weread-yourname-a8f3k"
export BARK_KEY="your-bark-key-here"
npx weread-selenium-cli run
```

#### Docker 运行

```bash
docker run --rm --name user-read \
  -v $HOME/weread-challenge/user/.weread:/app/.weread \
  -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 \
  -e WEREAD_DURATION=180 \
  -e NTFY_TOPIC="weread-yourname-a8f3k" \
  -e BARK_KEY="your-bark-key-here" \
  docker.io/lonemoonspace/weread-challenge:latest
```

#### Crontab 定时任务示例

```bash
# ntfy 推送示例（Android）
00 01 * * * docker run --rm --name user1-read -v /home/test/weread-challenge/user1/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=180 -e WEREAD_USER=user1 -e WEREAD_SELECTION=2 -e NTFY_TOPIC=weread-yourname-a8f3k docker.io/lonemoonspace/weread-challenge:latest

# Bark 推送示例（iOS）
00 01 * * * docker run --rm --name user1-read -v /home/test/weread-challenge/user1/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=180 -e WEREAD_USER=user1 -e WEREAD_SELECTION=2 -e BARK_KEY=your-bark-key-here docker.io/lonemoonspace/weread-challenge:latest
```

### 注意事项

- Bark 推送依赖 iOS 设备上的 Bark App，请确保设备已安装并配置正确
- ntfy 推送依赖 ntfy App 订阅了 `NTFY_TOPIC` 对应的主题；公共服务器上主题名等于密码，请使用随机后缀
- 只需设置 `NTFY_TOPIC` 或 `BARK_KEY` 即可启用对应渠道，无需额外开关
- 支持自定义服务器：ntfy 使用 `NTFY_SERVER`，Bark 使用 `BARK_SERVER`
- 当脚本检测到微信读书登录二维码并解析出登录链接时，会通过已启用的推送渠道发送链接（点击即可打开）

## 登录链接推送配置（Bark + ntfy + 邮件）

脚本在检测到新的微信读书登录链接后，会自动推送：

- ntfy：配置 `NTFY_TOPIC` 即可启用（Android 推荐）
- Bark：配置 `BARK_KEY` 即可启用
- 邮件：`ENABLE_EMAIL=true` 且配置完整 SMTP 参数后启用

Docker Compose `environment` 示例：

```yaml
environment:
  - WEREAD_REMOTE_BROWSER=http://selenium:4444
  - WEREAD_DURATION=15
  - WEREAD_SCREENSHOT=false
  - WEREAD_USER=your-user
  - WEREAD_SELECTION=-1
  - ENABLE_EMAIL=true
  - EMAIL_SMTP=smtp.mail.me.com
  - EMAIL_USER=your-email@icloud.com
  - EMAIL_PASS=your-app-password
  - EMAIL_PORT=587
  - EMAIL_TO=your-receiver@example.com
  - NTFY_TOPIC=weread-yourname-a8f3k
  - BARK_KEY=your-bark-key
  - DEFAULT_BOOK_URL=https://weread.qq.com/web/reader/276323e0813ab90a5g0144d7
```

说明：

- 邮件推送登录链接依赖 `EMAIL_SMTP`、`EMAIL_USER`、`EMAIL_PASS`、`EMAIL_TO`
- 若二维码刷新后登录链接变化，会推送新链接
- 同一个登录链接只推送一次，避免重复提醒

## 多用户支持

```bash
# 创建桥接网络
docker network create weread-challenge-net

# 启动浏览器服务
docker run --restart always -d --name selenium-live \
  --shm-size="2g" \
  --network weread-challenge-net \
  --hostname selenium-live \
  -p 4444:4444 \
  -p 7900:7900 \
  -e SE_ENABLE_TRACING=false \
  -e SE_BIND_HOST=false \
  -e SE_JAVA_DISABLE_HOSTNAME_VERIFICATION=false \
  -e SE_NODE_MAX_INSTANCES=10 \
  -e SE_NODE_MAX_SESSIONS=10 \
  -e SE_NODE_OVERRIDE_MAX_SESSIONS=true \
  selenium/standalone-chromium:latest

# 设置用户名
WEREAD_USER1="user1"
WEREAD_USER2="user2"

# 创建用户数据目录
mkdir -p $HOME/weread-challenge/$WEREAD_USER1/.weread
mkdir -p $HOME/weread-challenge/$WEREAD_USER2/.weread

# 添加定时任务（每隔6个小时，阅读 68 分钟）
# 首次启动后需微信扫描二维码登录，二维码保存在：
# $HOME/weread-challenge/${WEREAD_USER1}/.weread/login.png
# $HOME/weread-challenge/${WEREAD_USER2}/.weread/login.png
(crontab -l 2>/dev/null; echo "00 */6 * * * docker run --rm --name ${WEREAD_USER1}-read -v $HOME/weread-challenge/${WEREAD_USER1}/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=68 -e WEREAD_USER=${WEREAD_USER1} docker.io/lonemoonspace/weread-challenge:latest") | crontab -
(crontab -l 2>/dev/null; echo "00 */6 * * * docker run --rm --name ${WEREAD_USER2}-read -v $HOME/weread-challenge/${WEREAD_USER2}/.weread:/app/.weread --network weread-challenge-net -e WEREAD_REMOTE_BROWSER=http://selenium-live:4444 -e WEREAD_DURATION=68 -e WEREAD_USER=${WEREAD_USER2} docker.io/lonemoonspace/weread-challenge:latest") | crontab -
```

## 可配置项

| 环境变量                | 默认值                                                    | 可选值                                | 说明                                                                                            |
| ----------------------- | --------------------------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `WEREAD_USER`           | `weread-default`                                          | -                                     | 用户标识                                                                                        |
| `WEREAD_REMOTE_BROWSER` | ""                                                        | -                                     | 远程浏览器地址                                                                                  |
| `WEREAD_DURATION`       | `10`                                                      | -                                     | 阅读时长                                                                                        |
| `WEREAD_SPEED`          | `slow`                                                    | `slow,normal,fast`                    | 阅读速度                                                                                        |
| `WEREAD_SCREENSHOT`     | `true`                                                    | `true,false`                          | 阅读期间每分钟截图                                                                              |
| `WEREAD_SELECTION`      | `2`                                                       | `-1,0,1,2,3,4`                        | 选书方式，`-1` 直接读取 `DEFAULT_BOOK_URL`，`0` 随机选择第 1 到 4 本，`1-4` 表示书架中的第 N 本 |
| `DEFAULT_BOOK_URL`      | https://weread.qq.com/web/reader/276323e0813ab90a5g0144d7 | -                                     | 默认阅读链接，`WEREAD_SELECTION=-1` 或书架中未找到可点击书籍时会直接使用                        |
| `WEREAD_BROWSER`        | `chrome`                                                  | `chrome,MicrosoftEdge,firefox,safari` | 浏览器                                                                                          |
| `ENABLE_EMAIL`          | `false`                                                   | `true,false`                          | 邮件通知                                                                                        |
| `EMAIL_SMTP`            | ""                                                        | -                                     | 邮箱 SMTP 服务器                                                                                |
| `EMAIL_PORT`            | `465`                                                     | `25,465,587`                          | 邮箱 SMTP 端口                                                                                  |
| `EMAIL_USER`            | ""                                                        | -                                     | 邮箱用户名                                                                                      |
| `EMAIL_PASS`            | ""                                                        | -                                     | 邮箱密码                                                                                        |
| `EMAIL_FROM`            | ""                                                        | -                                     | 发件人                                                                                          |
| `EMAIL_TO`              | ""                                                        | -                                     | 收件人                                                                                          |
| `BARK_KEY`              | ""                                                        | -                                     | Bark 推送密钥（iOS）                                                                            |
| `BARK_SERVER`           | `https://api.day.app`                                     | -                                     | Bark 服务器地址                                                                                 |
| `NTFY_TOPIC`            | ""                                                        | -                                     | ntfy 主题（等价于密码），配置后启用 ntfy 推送                                                    |
| `NTFY_SERVER`           | `https://ntfy.sh`                                         | -                                     | ntfy 服务器地址，自建服务时填写                                                                  |
| `NTFY_TOKEN`            | ""                                                        | -                                     | ntfy Access Token，与用户名密码二选一                                                            |
| `NTFY_USERNAME`         | ""                                                        | -                                     | ntfy Basic Auth 用户名                                                                          |
| `NTFY_PASSWORD`         | ""                                                        | -                                     | ntfy Basic Auth 密码                                                                            |
| `NTFY_PRIORITY`         | ""                                                        | `1-5,min,low,default,high,urgent`     | 覆盖 ntfy 默认优先级映射，发送前统一归一化为 1-5 数字                                           |
| `NTFY_TAGS`             | ""                                                        | -                                     | 覆盖 ntfy 默认标签，逗号分隔                                                                     |
| `WEREAD_AGREE_TERMS`    | `false`                                                   | `true,false`                          | 是否向上游统计服务上报使用信息，见[隐私政策](#隐私政策)                                          |

## 容器架构支持

本仓库**只构建并发布 `linux/arm64`**（树莓派等 ARM 开发板）。`amd64` 不再构建，也不会推到镜像仓库；
需要 `amd64` 请使用上游仓库。

`docker-compose.yml` 使用的 `selenium/standalone-chromium` 已提供 `linux/arm64`,
所以 `arm64` 可以直接使用 `docker compose` 方式运行。

```bash
# 发布 arm64 镜像, push 默认就是 linux/arm64
npm run docker:image:push -- --tag latest

# 只在本机构建, 不推送
npm run docker:image:build

# 手工使用 buildx
docker buildx build --platform linux/arm64 -t docker.io/lonemoonspace/weread-challenge:latest -f Dockerfile --push .
```

> 在 `amd64` 机器上构建 `arm64` 镜像需要 QEMU/binfmt 模拟, 例如
> `docker run --privileged --rm tonistiigi/binfmt --install arm64`; 在 ARM 设备或 GitHub Actions
> 上不需要额外配置. 推送单平台镜像不要求 `docker-container` builder, 默认的 `docker` driver 即可.

## 注意事项

- **时长计算**：28 日刷满 30 小时，需每日至少 65 分钟，而不是每日 60 分钟
- **统计误差**：微信读书统计可能会漏数分钟，期望每日获得 65 分钟，建议调整阅读时长到 68 分钟
- **登录有效期**：网页扫码登录 cookies 有效期为 30 天，实测登录一次可以长期有效
- **邮件通知**：邮件通知可能被识别为垃圾邮件，建议在收件方添加白名单
- **使用声明**：本项目仅供学习交流使用，请勿用于商业用途，请勿用于违法用途
- **侵权处理**：如存在可能的侵权，请在本仓库 [提交 issue](https://github.com/lonemoonspace/weread-challenge-selenium/issues) 联系，本分支会立即删除

## 隐私政策

- **默认不上报**：本分支默认 `WEREAD_AGREE_TERMS=false`，不会向任何统计服务发送数据；本分支维护者不收集任何使用者信息
- **可选上报**：设置 `WEREAD_AGREE_TERMS=true` 后，登录成功和异常退出时会向**上游作者**的统计服务 `https://weread-challenge.techfetch.dev/logs` 上报以下信息，该服务与本分支无关：
  - 微信读书 `cookies` 中的部分字段：`wr_vid`、`wr_name`、`wr_avatar`、`wr_gender`、`wr_gid`、`wr_localvid`、`wr_rt`
  - 运行信息：浏览器类型、操作系统类别、阅读时长设置、是否启用邮件、版本号、异常退出原因
- **风险提示**
  - `cookies` 可用于微信读书网页登录，开启上报前请确认你信任接收方
  - 腾讯保护机制确保异常登录时，手机客户端将收到风险提示，可在手机客户端 `设置` -> `登录设备` 中确认登录设备
  - 本工具纯 JavaScript 实现，第三方可以继续开发。即使信任本工具，也应在使用自动化工具时，经常确认登录设备，避免书架被恶意操作

## 参考

- npm 包（上游发布，非本分支）: [weread-selenium-cli](https://www.npmjs.com/package/weread-selenium-cli)
- 上游开源地址: [https://github.com/jqknono/weread-challenge-selenium](https://github.com/jqknono/weread-challenge-selenium)
- 本分支(arm64): [https://github.com/lonemoonspace/weread-challenge-selenium](https://github.com/lonemoonspace/weread-challenge-selenium)
- 上游统计: [https://weread-challenge.techfetch.dev](https://weread-challenge.techfetch.dev)
- 上游文章: [https://blog.techfetch.dev](https://blog.techfetch.dev/blog/2024/12/05/%E5%BE%AE%E4%BF%A1%E8%AF%BB%E4%B9%A6%E8%87%AA%E5%8A%A8%E6%89%93%E5%8D%A1%E5%88%B7%E6%97%B6%E9%95%BF/)
