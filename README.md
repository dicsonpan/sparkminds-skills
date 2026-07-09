# sparkminds-skills

SparkMinds 技能库管理器 — 从 api.sparkminds.io 下载、上传、更新、删除 AI 技能包。

## 这是什么

这是一个 [AI Agent Skill](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)，用于管理部署在 `api.sparkminds.io`（Cloudflare R2 存储）上的 AI 技能包。技能以 **zip 压缩包**形式存储，每个 zip 包含一个完整的技能目录（含 `SKILL.md` 及附属文件）。

支持的操作：

- 查看服务器上的技能列表
- 下载技能到本地（服务器 → 本地）
- 上传新技能到服务器（本地 → 服务器）
- 更新已有技能（本地 → 服务器，覆盖）
- 删除服务器上的技能

## 安装

### 方式一：通过 GitHub 仓库安装

将本仓库克隆到你的 AI Agent 技能目录下：

```bash
# DuMate 用户
git clone https://github.com/dicsonpan/sparkminds-skills.git "~/Library/Application Support/qianfan-desktop-app/qianfan_desk_xdg/<workspace-id>/data/skills/user/sparkminds-skills"

# Trae CN 用户
git clone https://github.com/dicsonpan/sparkminds-skills.git ~/.trae-cn/skills/sparkminds-skills

# Cursor 用户
git clone https://github.com/dicsonpan/sparkminds-skills.git ~/.cursor/skills/sparkminds-skills
```

> ⚠️ 路径中的 `<workspace-id>` 需替换为你的实际 workspace ID。

### 方式二：通过 sparkminds-skills 自身安装（已安装其他实例）

如果你已经在另一个 App 中安装了 `sparkminds-skills`，可以直接用它在当前 App 中安装自身：

```
下载技能 sparkminds-skills
```

## 首次使用配置

安装后首次使用时，AI Agent 会询问你的 SparkMinds 账号密码。验证成功后凭证会保存到本地 `.credentials.json`（已加入 `.gitignore`，不会被提交）。

如果你需要 ADMIN 权限（上传/更新/删除），请联系管理员开通。

## 使用方式

安装完成后，在你的 AI Agent 中用自然语言触发：

```
查看技能列表
下载 sales-lead-tracker
下载所有技能
上传 sparkminds-skills
更新服务器上的 student-growth-tracker
同步 sales-lead-tracker 到服务器
删除 test-skill
更新凭证
```

## 技能目录结构

```
sparkminds-skills/
├── SKILL.md          ← 技能定义文件（核心）
├── _meta.json         ← 内容哈希
├── .gitignore         ← 排除凭证文件
└── README.md          ← 本文件
```

> 注意：`.credentials.json` 在首次使用后自动生成，包含账号密码，**不会**被提交到 GitHub（已加入 `.gitignore`）。

## 技术细节

- **服务器**：`https://api.sparkminds.io`（Cloudflare R2 存储）
- **认证**：Bearer Token，每次操作前重新登录
- **存储格式**：zip 压缩包，保留完整目录结构
- **多 App 适配**：自动检测运行环境（DuMate / Trae CN / Cursor），使用对应的技能存储路径
- **安全**：打包上传时自动排除 `.credentials.json`，上传后下载回传校验 MD5

## 版本

当前版本：**v2.0.0**（2026-07-01）

详见 [SKILL.md](./SKILL.md) 中的更新日志。

## 作者

潘江浩 · [创智教育 SparkMinds](https://sparkminds.cn)
