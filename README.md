<h2>📖 简介</h2>

<img src="./images/WeChat-OpenCode-Bridge.svg" alt="WX-OpenCode" width="360" height="300" align="right">

<p><strong>WX-OpenCode</strong> 是一个基于 Node.js 的桥接服务，让你通过微信使用 OpenCode AI。</p>
<ul>
  <li>🔗 连接微信个人账号与 OpenCode</li>
  <li>💬 发送微信消息，获取 AI 回复</li>
  <li>🔄 创建、管理和切换多个会话</li>
  <li>📱 扫码绑定，快速开始使用</li>
</ul>

<!-- <br clear="right"> -->

<p align="left">
  <img src="https://img.shields.io/badge/TypeScript-5.9.3-blue?logo=typescript&amp;logoColor=white" alt="TypeScript 5.9.3" height="16">
  <img src="https://img.shields.io/badge/Node.js-24.14.1-green?logo=node.js&amp;logoColor=white" alt="Node.js 24.14.1" height="16">
  <img src="https://img.shields.io/badge/npm-11.1.0-red?logo=npm&amp;logoColor=white" alt="npm 11.1.0" height="16">
  <img src="https://img.shields.io/badge/License-MIT-purple" alt="MIT License" height="16">
</p>

## 🏗️ 项目架构
```mermaid
flowchart LR
    %% 节点定义
    A["📱 微信客户端"]:::wechat
    B["🌉 WX-OpenCode"]:::bridge
    C["🤖 OpenCode Server"]:::opencode
    D["💾 本地存储<br/>~/.WeChat-OpenCode-Bridge/"]:::storage

    %% 连接线
    A -->|消息/指令| B
    B -->|API 请求| C
    C -->|OpenCode 回复| B
    B -->|消息/回复| A
    B -->|读写操作| D

    %% 样式
    classDef wechat fill:#00B96B,stroke:#095028,stroke-width:3px,color:#fff,font-weight:bold;
    classDef bridge fill:#1890FF,stroke:#003EB3,stroke-width:3px,color:#fff,font-weight:bold;
    classDef opencode fill:#F59E0B,stroke:#B45309,stroke-width:3px,color:#fff,font-weight:bold;
    classDef storage fill:#6B7280,stroke:#374151,stroke-width:3px,color:#fff,font-weight:bold;
```

```
WX-OpenCode/
├── src/
│   ├── index.ts              # CLI 入口（setup、start、status 命令）
│   ├── logger.ts             # 结构化日志单例
│   ├── constants.ts          # 共享常量（路径、URL、限制）
│   ├── config.ts             # 配置加载/保存
│   ├── commands/
│   │   └── router.ts         # 斜杠命令路由（/help、/new、/sessions 等）
│   ├── opencode/
│   │   └── client.ts         # OpenCode REST API 客户端
│   ├── store/
│   │   ├── account.ts        # 账号凭证持久化
│   │   └── session.ts        # 会话状态 + 聊天历史持久化
│   └── wechat/
│       ├── types.ts          # WeChat ilink API 类型定义
│       ├── api.ts            # WeChat API 客户端
│       ├── login.ts          # 二维码登录流程
│       ├── fetch-helper.ts   # fetch() 包装器（重试 + 超时）
│       └── monitor.ts        # 长轮询消息监听器
├── dist/                     # 编译输出目录
├── node_modules/             # 依赖包   
├── package.json              # 项目信息和依赖
├── tsconfig.json             # TypeScript 配置
└── AGENTS.md                 # 开发者文档
```


## ✅ 前提条件
- **[Node.js](https://nodejs.org/)** 24+ 和 **[npm](https://www.npmjs.com/)** 11+
- 已安装并配置好 **[OpenCode](https://opencode.ai/)**
- **[微信](https://weixin.qq.com/)** 8.0.70+，账号支持 Clawbot


## 🚀 使用方法
### 1. 安装并绑定微信

```bash
git clone https://github.com/Nuyoahwjl/WX-OpenCode.git
cd WX-OpenCode
npm install
npm run setup
```

安装时会自动编译。首次使用时，用微信扫描终端中的二维码完成绑定。

### 2. 启动 OpenCode

另开一个终端，在你希望 OpenCode 操作的**工作目录**中运行：

```bash
opencode serve
```

### 3. 启动桥接服务

回到 **WX-OpenCode 仓库目录**的终端，运行：

```bash
npm start
```

保持两个终端运行，即可通过微信发送消息。以后使用只需执行第 2、3 步；按 `Ctrl+C` 停止对应服务。

## 📋 可用命令
| 命令 | 说明 |
|------|------|
| `npm run setup` | 扫码绑定微信（首次使用） |
| `npm start` | 启动桥接服务 |
| `npm run dev` | 开发模式（自动重新编译） |
| `npm run build` | 手动编译 TypeScript |
| `npm run status` | 查看当前绑定账号 |


## 🔦 微信快捷指令
| 指令 | 说明 | 示例 |
|------|------|------|
| `/help` | 显示帮助信息 | `/help` |
| `/new [标题]` | 创建新会话 | `/new 我的项目` |
| `/rename <标题>` | 重命名当前会话 | `/rename 新名字` |
| `/delete [标题]` | 删除会话 | `/delete 项目` 或 `/delete` |
| `/history [数量]` | 查看聊天历史 | `/history 10` |
| `/sessions` | 列出所有会话 | `/sessions` |
| `/switch <标题>` | 切换到指定会话 | `/switch 项目` |

<details>
<summary>指令详细说明（点击展开）</summary>

#### `/help`
显示所有可用指令的帮助信息。

#### `/new [标题]`
创建一个新的 OpenCode 会话。
- 有参数：使用 `WeChat: <标题>` 作为会话名
- 无参数：使用时间戳作为会话名（格式：`WeChat: YYYY-MM-DD-HH-mm-ss`）

#### `/rename <标题>`
重命名当前会话。会话名会自动添加 `WeChat: ` 前缀。

#### `/delete [标题]`
删除指定的会话（同时删除 OpenCode 服务器和本地记录）。
- 有参数：删除标题**包含**该关键词的会话（不区分大小写）
- 无参数：删除当前会话
- 删除当前会话后，请使用 `/new` 创建新会话或 `/switch` 切换到其他会话

#### `/history [数量]`
查看当前会话的聊天历史记录。
- 有参数：显示最近 N 条记录
- 无参数：显示最近 20 条记录

#### `/sessions`
列出 OpenCode 服务器上的所有会话，显示标题、ID、创建时间和当前会话标记。

#### `/switch <标题>`
切换到标题**包含**指定关键词的会话（不区分大小写）。
- 如果本地已有该会话的历史记录，会自动恢复
- 支持模糊匹配，例如 `/switch 项目` 会匹配到 `WeChat: 我的项目`

</details>

## 📁 数据目录
所有数据存储在用户主目录下的 `.WeChat-OpenCode-Bridge/` 文件夹中：
```
~/.WeChat-OpenCode-Bridge/
├── accounts/            # 账号凭证
│   └── xxxxxxx.json     # 微信绑定信息
├── sessions/            # 会话数据
│   └── xxxxxxx_xxx.json # 本地聊天历史
└── logs/                # 日志文件
```
每个会话文件包含：
- `sdkSessionId`：关联的 OpenCode 会话 ID
- `chatHistory`：本地聊天历史记录
- `state`：会话状态（idle/processing）


## 🛠️ 开发相关
```bash
npx tsc --noEmit  # 类型检查
npm run dev      # 监听代码变更并自动编译
```

开发模式仅重新编译代码；运行桥接服务使用 `npm start`。


## 🧩 演示
![demo-1](./images/demo-1.png)

![demo-2](./images/demo-2.png)


<div align="center">
    <img src="./images/demo-3.png" alt="demo-3" width="32%" style="display: inline-block;">
    <img src="./images/demo-4.png" alt="demo-4" width="32%" style="display: inline-block;">
    <img src="./images/demo-5.png" alt="demo-5" width="32%" style="display: inline-block;">
</div>

