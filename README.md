# Cloudflare Worker → Vercel Proxy Skill

[English](#english) · [简体中文](#简体中文)

---

<a id="english"></a>

## English

A reusable Agent Skill for placing an **isolated Vercel external-rewrite proxy** in front of a Cloudflare Worker. It is designed to preserve the original Worker, application repository, secrets, and any existing proxy project while preparing a Vercel custom domain entry point.

The Skill captures a production-verified pattern: use `/(.*)` → `$1` to cover the root path and all nested paths; do not add a static homepage that can shadow the origin; disable proxy caching for `/api/*`; validate NDJSON/SSE/model streaming before claiming success; and repair Vercel deployment blocks caused by unmatched Git author emails.

### Package layout

```text
cloudflare-worker-vercel-proxy-skill/
├── SKILL.md
├── README.md
└── package.json
```

`SKILL.md` is the entry point that Skill-compatible Agents should load.

### Remote installation

Clone this repository into the Agent's local Skill directory:

```bash
git clone https://github.com/Tan35/cloudflare-worker-vercel-proxy-skill.git \
  /home/ubuntu/skills/cloudflare-worker-vercel-proxy
```

After installation, the Agent should load:

```text
/home/ubuntu/skills/cloudflare-worker-vercel-proxy/SKILL.md
```

### Updating

Pull the latest version in an existing installation:

```bash
git -C /home/ubuntu/skills/cloudflare-worker-vercel-proxy pull --ff-only
```

### Use this Skill when

Use it for requests such as:

- Serve a `workers.dev` application through Vercel.
- Put a Vercel custom domain in front of a Cloudflare Worker.
- Add an isolated Vercel reverse proxy to a Cloudflare Worker application.
- Verify that Vercel preserves a Worker's NDJSON, SSE, or model streaming response.

This repository contains no keys, application source code, or Cloudflare runtime configuration.

---

<a id="简体中文"></a>

## 简体中文

这是一个可复用的 Agent Skill，用于在 Cloudflare Worker 前放置一个**隔离的 Vercel 外部重写代理**。它在为 Vercel 自定义域名提供入口的同时，保留原 Worker、应用仓库、密钥与已有代理项目不变。

该 Skill 固化了经过生产环境验证的模式：使用 `/(.*)` → `$1` 覆盖根路径与全部子路径；不添加会遮挡原站的静态首页；为 `/api/*` 禁用代理缓存；在宣称部署成功前验证 NDJSON、SSE 或模型流式响应；并处理 Git 作者邮箱未匹配而导致的 Vercel 部署阻断。

### 包结构

```text
cloudflare-worker-vercel-proxy-skill/
├── SKILL.md
├── README.md
└── package.json
```

`SKILL.md` 是兼容 Skill 格式的 Agent 应加载的入口文件。

### 远程安装

将仓库克隆到 Agent 的本地 Skill 目录：

```bash
git clone https://github.com/Tan35/cloudflare-worker-vercel-proxy-skill.git \
  /home/ubuntu/skills/cloudflare-worker-vercel-proxy
```

安装后，Agent 应加载：

```text
/home/ubuntu/skills/cloudflare-worker-vercel-proxy/SKILL.md
```

### 更新

在已安装目录中拉取最新版本：

```bash
git -C /home/ubuntu/skills/cloudflare-worker-vercel-proxy pull --ff-only
```

### 适用场景

当用户提出以下或等价需求时使用本 Skill：

- 将 `workers.dev` 应用通过 Vercel 对外提供；
- 用 Vercel 自定义域名访问 Cloudflare Worker；
- 为 Cloudflare Worker 应用增加隔离的 Vercel 反向代理；
- 验证 Vercel 是否完整保留 Worker 的 NDJSON、SSE 或模型流式响应。

该仓库不包含任何密钥、应用源代码或 Cloudflare 运行时配置。
