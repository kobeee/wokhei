# Wok Hei Master - iFlow 项目指南

## 项目简介

**Wok Hei Master** 是一款基于 Reddit Devvit 平台的"中国功夫烹饪"主题小游戏。

**一句话 Pitch**：感受"中国功夫烹饪"的火焰与节奏，别把炒饭撒在地上。

### 核心玩法

1. **备菜**：各种颜色的方块（代表米饭、鸡蛋、葱花、牛肉）掉进锅里
2. **颠勺 (The Toss)**：
   - 鼠标左键按住 = 压锅蓄力
   - 鼠标上划松开 = 颠起
   - 鼠标移动 = 接住
3. **控火 (Fire Control)**：
   - 食材在空中停留时间越长，冷却越快
   - 接触锅底时间越长，温度越高（有进度条，变红会焦）
   - 完美状态：食材在火苗上掠过，触发"镬气"特效（金光一闪 + WHOOSH 声）
4. **上菜**：倒进盘子，评分基于色泽、完整度、镬气值

### 文化"翻译"策略

- **Uncle Roger 风格 NPC**：穿白色背心的大爷师傅
  - 失败时："Haiyaa! You killed the rice!"
  - 成功时："Fuiyoh!"
- **声音反馈**：真实的铁锅撞击声和油脂爆裂声（ASMR 玩家最爱）

---

## 目录结构

```
/Users/elvis/Documents/codes/challenges/2026/Jan/wokhei/
├── docs/
│   ├── prd/
│   │   └── v1.0.md              # 产品需求文档
│   └── reddit/
│       └── devvit-proxy-setup.md # Devvit 代理配置指南
├── src/
│   ├── backend/                  # 后端代码（预留）
│   └── frontend/
│       └── wok-hei-master/       # Devvit Web 应用主目录
│           ├── src/
│           │   ├── client/       # 前端代码（iFrame 内执行）
│           │   ├── server/       # 后端代码（serverless 环境）
│           │   └── shared/       # 共享代码
│           ├── public/           # 静态资源
│           ├── devvit.json       # Devvit 配置
│           ├── vite.config.ts    # Vite 构建配置
│           └── package.json
└── IFLOW.md                      # 本文件
```

---

## 重要注意事项

### 网络代理（中国大陆必须）

由于 Reddit 在中国大陆被屏蔽，**所有涉及 Reddit 服务器的命令必须使用 `proxychains4`**：

```bash
# 正确示例
proxychains4 devvit login
proxychains4 devvit playtest
proxychains4 devvit upload
proxychains4 npm run dev
proxychains4 npm run deploy
proxychains4 npm run launch

# 错误示例（会超时或连接失败）
devvit login      # ❌
npm run dev       # ❌
```

### proxychains4 配置

配置文件：`/usr/local/etc/proxychains.conf`

```ini
[ProxyList]
http    127.0.0.1 7890    # 替换为你的代理地址
```

### Node.js 版本要求

Devvit 需要 **Node.js v22.2.0+**，推荐使用 nvm 管理：

```bash
nvm install 22
nvm use 22
```

---

## Devvit Web 应用指南

> 以下内容来自 `src/frontend/wok-hei-master/AGENTS.md`

### 技术栈

- **Frontend**: React 19, Tailwind CSS 4, Vite
- **Backend**: Node.js v22 serverless environment (Devvit), Hono, TRPC
- **Communication**: tRPC v11 for end-to-end type safety
- **Testing**: Vitest

### 架构说明

> 以下路径相对于 `src/frontend/wok-hei-master/` 目录

```
wok-hei-master/
├── src/
│   ├── server/       # 后端代码（serverless 环境）
│   ├── client/       # 前端代码（iFrame 内执行）
│   └── shared/       # 共享代码
├── public/           # 静态资源
└── devvit.json       # Devvit 配置
```

- `src/server/`：**后端代码**。运行在安全的 serverless 环境中。
  - `trpc.ts`：定义 API router 和 procedures
  - `index.ts`：服务器入口点（Hono app）
  - 通过 `@devvit/web/server` 访问 `redis`、`reddit` 和 `context`
- `src/client/`：**前端代码**。在 reddit.com 的 iFrame 内执行。
  - 入口点定义在 `devvit.json` 中：
    - `game.html`：主入口（展开视图）
    - `splash.html`：初始入口（内联视图，显示在 feed 中，保持轻量）
    - `trpc.ts`：tRPC 客户端实例
- `src/shared/`：**共享代码**。客户端和服务端共用

### 数据获取（tRPC）

项目使用 tRPC 进行客户端-服务端通信：

1. **定义 Procedure**：在 `src/server/trpc.ts` 中添加 query 或 mutation
2. **客户端调用**：在 React 组件中使用 `trpc.procedureName.query()` 或 `.mutate()`

### 前端规则

- 使用 `navigateTo`（来自 `@devvit/web/client`）代替 `window.location` 或 `window.assign`
- 使用 `showToast` 或 `showForm`（来自 `@devvit/web/client`）代替 `window.alert`
- 文件下载：使用 clipboard API 配合 `showToast` 确认
- 不支持：Geolocation、camera、microphone、notifications Web APIs
- HTML 文件中禁止内联 script 标签，使用单独的 js/ts 文件

### 常用命令

```bash
# 开发（需要代理）
proxychains4 npm run dev              # 启动 playtest
proxychains4 npm run deploy           # 部署到 Reddit
proxychains4 npm run launch           # 部署并发布

# 本地命令（无需代理）
npm run type-check                    # TypeScript 类型检查
npm run lint                          # ESLint 检查
npm run test -- my-file-name          # 运行测试
npm run prettier                      # 格式化代码
```

### 代码风格

- TypeScript 类型优先使用 `type` 而非 `interface`
- 优先使用命名导出而非默认导出
- 不要强制转换 TypeScript 类型

### 全局规则

- 项目配置为 Devvit Web，**不要使用** `@devvit/public-api` 或 blocks 相关代码
- 添加新的菜单项 action 时，确保在 `devvit.json` 中添加对应的映射

---

## 参考链接

- [Devvit 官方文档](https://developers.reddit.com/docs/llms.txt)
- [Devvit Template Library](https://developers.reddit.com/docs/examples/template-library)
- [Devvit Web 概览](https://developers.reddit.com/docs/capabilities/devvit-web/devvit_web_overview)
- [GitHub: devvit-template-threejs](https://github.com/reddit/devvit-template-threejs)
