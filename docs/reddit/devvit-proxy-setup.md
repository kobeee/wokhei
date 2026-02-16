# Devvit 代理配置指南

## 问题背景

在中国大陆使用 Devvit CLI 时，由于 Reddit 被屏蔽，`devvit login` 会报错：

```
TypeError: fetch failed
Failed to login to reddit
```

Node.js 原生 `fetch` 不会自动使用 `HTTP_PROXY` 环境变量，因此需要额外配置。

## 解决方案

使用 `proxychains-ng` 强制所有网络请求走代理。

### 1. 安装 proxychains-ng

```bash
brew install proxychains-ng
```

### 2. 配置代理

编辑配置文件 `/usr/local/etc/proxychains.conf`：

```bash
# 找到 [ProxyList] 部分，修改为：
[ProxyList]
http    127.0.0.1 7890
```

将 `127.0.0.1:7890` 替换为你的代理地址（Clash 默认端口为 7890）。

### 3. 使用代理运行命令

```bash
proxychains4 devvit login
proxychains4 devvit new --template blocks-post
proxychains4 devvit upload
proxychains4 devvit playtest <subreddit>
```

## 相关文件位置

| 文件 | 路径 | 说明 |
|------|------|------|
| proxychains 配置 | `/usr/local/etc/proxychains.conf` | 代理设置 |
| Devvit token | `~/.devvit/token` | 登录凭证 |
| Devvit 全局安装 | `~/.nvm/versions/node/v22.22.0/lib/node_modules/devvit/` | CLI 代码 |

## 常见问题

### Q: 端口 65010 被占用

```bash
# 查找并杀掉占用进程
lsof -ti :65010 | xargs kill -9
```

### Q: 登录码过期

使用 `npm create devvit@latest <code>` 方式的一次性授权码会很快过期。

**推荐方式**：全局安装 CLI 后用 `devvit login` 登录：

```bash
npm install -g devvit
proxychains4 devvit login
```

### Q: Node.js 版本要求

Devvit 需要 Node.js v22.2.0+，推荐使用 nvm 管理：

```bash
nvm install 22
nvm use 22
```

## 快速参考命令

```bash
# 安装 CLI
npm install -g devvit

# 登录（需要代理）
proxychains4 devvit login

# 创建项目
proxychains4 devvit new --template blocks-post <project-name>

# 上传应用
proxychains4 devvit upload

# 本地测试
proxychains4 devvit playtest r/<your-test-subreddit>
```

## 代理环境变量（备选方案）

如果不想用 proxychains，可以尝试以下方法（可能不适用于所有情况）：

```bash
# 设置环境变量（对 curl 有效，对 Node.js fetch 无效）
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890

# 验证代理工作
curl -x http://127.0.0.1:7890 https://www.reddit.com
```

## 参考链接

- [Devvit 官方文档](https://developers.reddit.com/docs/quickstart)
- [proxychains-ng GitHub](https://github.com/rofl0r/proxychains-ng)
