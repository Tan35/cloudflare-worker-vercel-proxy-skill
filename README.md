# Cloudflare Worker → Vercel Proxy Skill

一个可复用的 Agent Skill，用于在**不修改原 Cloudflare Worker、原应用仓库或已有代理项目**的前提下，新建隔离的 Vercel 外部重写代理，并为自定义域名做好准备。

该 Skill 固化了经过实际验证的代理模式：使用 `/(.*)` → `$1` 覆盖根路径和全部子路径；不创建会遮挡根路由的静态首页；禁止缓存 `/api/*`；在包含 NDJSON/SSE/模型输出的应用中进行真实流式验收；并在 Vercel 因 Git 作者邮箱不匹配而阻断部署时提供安全修复流程。

## 包结构

```text
cloudflare-worker-vercel-proxy-skill/
├── SKILL.md
└── README.md
```

`SKILL.md` 是可由支持 Manus Skill 格式的 Agent 直接加载的入口文件。

## 远程安装

将仓库克隆到 Agent 的本地 Skill 目录并使用仓库名作为目录名：

```bash
git clone https://github.com/Tan35/cloudflare-worker-vercel-proxy-skill.git \
  /home/ubuntu/skills/cloudflare-worker-vercel-proxy
```

私有仓库需要在目标环境中先完成对该 GitHub 账户的访问授权。安装后，Agent 应读取：

```text
/home/ubuntu/skills/cloudflare-worker-vercel-proxy/SKILL.md
```

## 更新

在已安装目录中拉取最新版本：

```bash
git -C /home/ubuntu/skills/cloudflare-worker-vercel-proxy pull --ff-only
```

## 适用场景

当用户提出以下或等价需求时使用本 Skill：

- 将 `workers.dev` 地址通过 Vercel 对外提供；
- 用 Vercel 自定义域名访问 Cloudflare Worker 应用；
- 为 Cloudflare Worker 添加隔离的 Vercel 反向代理；
- 验证 Vercel 是否会破坏 Worker 的 NDJSON、SSE 或模型流式响应。

该仓库不包含任何密钥、应用源代码或 Cloudflare 运行时配置。
