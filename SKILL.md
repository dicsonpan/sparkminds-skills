---
name: sparkminds-skills
description: SparkMinds 技能库管理器。从 api.sparkminds.io 下载、上传、更新、删除技能文件。当用户提到"下载技能"、"上传技能"、"更新技能"、"查看技能列表"、"删除技能"、"管理技能库"、"同步技能"时触发。首次使用询问账号密码并保存到本地，后续自动读取凭证登录。技能以 zip 包形式存储，支持完整目录结构。
---

# SparkMinds 技能库管理器

## 定位

管理 api.sparkminds.io 服务器（Cloudflare R2 存储）上的 AI 技能包，支持查看列表、下载、上传、更新、删除。技能以 **zip 压缩包**形式存储在服务器上，每个 zip 包含一个完整的技能目录（含 SKILL.md 及附属文件）。首次使用时询问用户账号密码并保存到本地凭证文件，后续调用自动读取凭证并登录获取 Token。

**核心价值：**
- 完整目录打包：技能以 zip 包形式存储，保留完整目录结构（SKILL.md + 模板 + 附属文件）
- 一站式管理：列表、下载、上传、更新、删除，一个技能全搞定
- 凭证记忆：首次输入账号密码后自动保存到本地文件，后续无需重复输入
- 安全校验：上传后下载回传校验 MD5，下载后校验 MD5，确保数据一致
- 多 App 适配：自动检测运行环境，使用对应 App 的技能存储路径

---

## 触发条件

**查看技能列表：**
- "查看技能列表"
- "有哪些技能"
- "列出技能"
- "服务器上有哪些技能"

**下载技能（从服务器拉取到本地）：**
- "下载技能"
- "下载 xxx 技能"
- "从服务器拉取技能"
- "下载所有技能"

**上传技能（推送本地到服务器）：**
- "上传技能"
- "上传 xxx 技能到服务器"
- "发布技能"
- "把 xxx 技能传上去"

**更新技能（推送本地到服务器，覆盖已有）：**
- "更新技能"
- "更新服务器上的 xxx 技能"
- "同步技能到服务器"
- "把本地修改同步到服务器"

**更新本地技能（从服务器拉取，覆盖本地）：**
- "更新本地 xxx 技能"
- "从服务器更新 xxx"

**删除技能：**
- "删除技能"
- "删除服务器上的 xxx 技能"

**凭证管理：**
- "更新凭证"
- "重新登录"
- "修改账号密码"
- "忘记密码了"

---

## API 配置

**服务器地址：** `https://api.sparkminds.io`

**认证方式：** Bearer Token（所有 `/api/skills` 路由需要认证，上传/更新/删除需要 `ADMIN` 角色)

---

## 核心概念：zip 包存储模式

### 存储格式

服务器上的每个技能以 **zip 压缩包**形式存储，文件名为 `<skill-name>.zip`。

zip 包内部保留完整的技能目录结构：

```
<skill-name>.zip
└── <skill-name>/
    ├── SKILL.md          ← 技能定义文件（必须）
    ├── templates/         ← 模板文件（可选）
    │   └── xxx.md
    └── ...                ← 其他附属文件（可选）
```

### 本地 ↔ 服务器映射规则

| 本地 | 服务器 |
|------|--------|
| `SKILLS_DIR/<skill-name>/`（整个目录） | `<skill-name>.zip`（zip 包） |

示例：
- 本地 `sales-lead-tracker/` 目录 ↔ 服务器 `sales-lead-tracker.zip`
- 本地 `student-growth-tracker/` 目录 ↔ 服务器 `student-growth-tracker.zip`

### API 端点

| 端点 | 方法 | 说明 | 权限要求 |
|------|------|------|----------|
| `/api/auth/login` | POST | 登录获取 Token | 无需认证 |
| `/api/skills` | GET | 获取技能列表 | 需认证 |
| `/api/skills/{filename}` | GET | 下载指定技能 zip 包 | 需认证 |
| `/api/skills` | POST | 上传新技能 zip 包（multipart/form-data） | 需 ADMIN |
| `/api/skills/{filename}` | PUT | 更新已有技能 zip 包（multipart/form-data） | 需 ADMIN |
| `/api/skills/{filename}` | DELETE | 删除技能 | 需 ADMIN |

> **注意：** `{filename}` 为 zip 包文件名，如 `sales-lead-tracker.zip`。

---

## 🔴 运行环境检测（每次操作必须执行！）

> **强制规则：** 本技能在多个 AI App 中运行（Trae CN、DuMate 等），不同 App 的技能存储路径完全不同。**每次执行上传、下载、更新等操作前，必须先按本节规则确定技能根目录的绝对路径，并将该路径替换到所有后续命令中。严禁硬编码任何 App 的路径！**

### 如何确定技能根目录

按以下优先级依次检测，**命中即停止**：

**优先级 1（最高）：读取 `<env>` 环境信息中的「技能安装目录」字段**

当本技能被加载时，系统会提供 `<env>` 环境信息，其中包含「技能安装目录」字段。若该字段存在，**直接使用其完整值作为技能根目录**。

例如 DuMate 环境中，该字段值为：
```
/Users/dicsonpan/Library/Application Support/qianfan-desktop-app/qianfan_desk_xdg/cb18f6c82d304ab3860ba8248226a660/data/skills/user/
```

**优先级 2：检查目录存在性**

若 `<env>` 中无「技能安装目录」字段，依次检查以下目录是否存在，取第一个存在的：

| App | 检查的目录 | 技能根目录 |
|-----|----------|-----------|
| Trae CN | `~/.trae-cn/skills/` | `~/.trae-cn/skills/` |
| Cursor | `~/.cursor/skills/` | `~/.cursor/skills/` |

**优先级 3：询问用户**

若以上均无法确定，询问用户："请提供技能存储目录的绝对路径："

### 确定路径后如何使用

一旦确定技能根目录的绝对路径（下文称为 `SKILLS_DIR`），**必须在所有命令中直接写入该绝对路径**。

**❌ 错误写法**（Shell 中 `SKILLS_DIR` 未定义，会展开为空）：
```bash
zip -r skill.zip ${SKILLS_DIR}/skill-name/
```

**✅ 正确写法**（直接写入绝对路径）：
```bash
# Trae CN 环境
cd /Users/dicsonpan/.trae-cn/skills/ && zip -r /tmp/skill.zip skill-name/

# DuMate 环境（路径含空格，需引号）
SKILLS_DIR="/Users/dicsonpan/Library/Application Support/qianfan-desktop-app/qianfan_desk_xdg/cb18f6c82d304ab3860ba8248226a660/data/skills/user/"
cd "$SKILLS_DIR" && zip -r /tmp/skill.zip skill-name/
```

### 关键路径推导

| 用途 | 路径 |
|------|------|
| 本技能目录 | `SKILLS_DIR/sparkminds-skills/` |
| 凭证文件 | `SKILLS_DIR/sparkminds-skills/.credentials.json` |
| 某技能目录 | `SKILLS_DIR/<skill-name>/` |
| 临时 zip 缓存 | `/tmp/<skill-name>.zip` |

---

## 凭证管理（核心流程）

### 流程总览

```
技能被触发
    ↓
按「运行环境检测」确定 SKILLS_DIR 的绝对路径
    ↓
检查凭证文件是否存在？（SKILLS_DIR/sparkminds-skills/.credentials.json）
    ├── 存在 → 读取 username + password → 登录获取 Token → 执行操作
    └── 不存在 → 询问用户账号密码 → 登录验证 → 保存凭证文件 → 执行操作
                                                    ↓
                                              若登录失败 → 提示重新输入
```

### 首次使用：询问并保存凭证

**交互流程：**
```
用户：下载技能 sales-lead-tracker
AI：检测到您尚未配置 SparkMinds 账号，请提供登录凭证：
    用户名：？
    密码：？

用户：dicson，P@ssw0rd_api
AI：正在验证凭证...
    ✅ 登录成功！角色：ADMIN
    凭证已保存到本地，后续无需重复输入。
    
    正在执行您的请求...
```

**第一步：登录验证**
```bash
curl -s -X POST https://api.sparkminds.io/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"<用户提供的用户名>","password":"<用户提供的密码>"}'
# 返回: {"success":true,"token":"<token>","role":"ADMIN 或 TEACHER"}
```

**第二步：验证成功后，保存凭证文件**

使用 Write 工具将以下 JSON 内容写入凭证文件路径 `SKILLS_DIR/sparkminds-skills/.credentials.json`（`SKILLS_DIR` 替换为实际绝对路径）：

```json
{
  "username": "<用户名>",
  "password": "<密码>",
  "role": "<登录返回的角色>",
  "saved_at": "<当前 UTC 时间，ISO 8601 格式>"
}
```

### 后续使用：自动读取凭证

每次技能被触发时，**必须**执行以下步骤：

1. **按「运行环境检测」确定 `SKILLS_DIR` 的绝对路径**
2. **读取凭证文件** `SKILLS_DIR/sparkminds-skills/.credentials.json`（用 Read 工具读取实际绝对路径）
3. **提取** `username` 和 `password` 字段
4. **调用登录接口**获取新的 Token（每次操作都重新登录，避免 Token 过期）
5. **使用 Token** 执行用户请求的操作

### 登录失败处理

若登录接口返回失败（凭证过期、密码修改等）：

```
AI：⚠️ 登录失败，凭证可能已失效。
    服务器返回：<错误信息>
    
    请重新提供登录凭证：
    用户名：？
    密码：？

用户：dicson，newpassword
AI：正在验证...
    ✅ 登录成功！凭证已更新。
    
    继续执行您的请求...
```

### 主动更新凭证

当用户说"更新凭证"、"重新登录"、"修改账号密码"时：

```
用户：更新凭证
AI：请提供新的登录凭证：
    用户名：？
    密码：？

用户：dicson，newpass
AI：正在验证...
    ✅ 登录成功！角色：ADMIN
    凭证已更新。
```

### 安全说明

- 凭证文件 `.credentials.json` 以点号开头（隐藏文件），存储在本地技能目录中
- **上传 zip 包时必须排除 `.credentials.json`**（见上传模块的排除规则）
- 仅在本机存储，不会上传到任何第三方服务
- 用户可随时通过"更新凭证"修改，或直接删除该文件来重置
- **注意：** 请勿将 `.credentials.json` 文件提交到 Git 等版本控制系统

---

## 功能模块

### 模块一：查看技能列表

**调用 `GET /api/skills`：**

```bash
curl -s https://api.sparkminds.io/api/skills \
  -H "Authorization: Bearer <token>"
```

**返回格式：**
```json
{
  "skills": [
    {
      "key": "sales-lead-tracker.zip",
      "size": 15234,
      "uploadedAt": "2026-07-01T03:54:01.849Z"
    },
    {
      "key": "student-growth-tracker.zip",
      "size": 18456,
      "uploadedAt": "2026-07-01T03:54:06.949Z"
    }
  ]
}
```

**交互示例：**
```
用户：查看技能列表
AI：正在获取技能列表...

    服务器技能库（共 2 个技能）：

    1. sales-lead-tracker.zip
       大小：14.9 KB
       上传时间：2026-07-01 11:54 (北京时间)

    2. student-growth-tracker.zip
       大小：18.0 KB
       上传时间：2026-07-01 11:54 (北京时间)

    输入"下载 <编号/名称>"可下载技能，输入"上传"可上传新技能。
```

**文件大小格式化：**
- < 1024 B → 显示 "X B"
- < 1,048,576 B → 显示 "X.X KB"
- ≥ 1,048,576 B → 显示 "X.X MB"

**时间格式化：**
服务器返回 UTC 时间，展示时转为用户本地时区。若用户时区为亚洲/上海（UTC+8）：
- `2026-07-01T03:54:01.849Z` → `2026-07-01 11:54 (北京时间)`

---

### 模块二：下载技能（从服务器拉取到本地）

> **下载 = 用服务器版本覆盖本地**：下载 zip → MD5 校验 → 删除本地旧目录 → 解压 → 删除 zip

**调用 `GET /api/skills/{filename}` 下载 zip 包：**

```bash
# 下载 zip 包到临时目录
curl -s https://api.sparkminds.io/api/skills/<skill-name>.zip \
  -H "Authorization: Bearer <token>" \
  -o /tmp/<skill-name>.zip
```

**完整下载流程（必须严格按顺序执行）：**

1. **确定路径**：按「运行环境检测」确定 `SKILLS_DIR` 的绝对路径
2. **下载 zip 包**：从服务器下载 `<skill-name>.zip` 到 `/tmp/<skill-name>.zip`
3. **MD5 校验**：计算下载的 zip 包 MD5 值，与服务器端对比（若服务器返回了 MD5）；若服务器未返回 MD5，则验证 zip 文件完整性（`unzip -t /tmp/<skill-name>.zip`）
4. **删除本地旧目录**：若本地已存在 `SKILLS_DIR/<skill-name>/` 目录，**整个删除**：
   ```bash
   rm -rf "SKILLS_DIR/<skill-name>/"
   ```
   > **注意：** 这会删除本地该技能目录下的所有文件。如果用户本地有未提交的修改，应先提示用户确认。
5. **解压 zip 包**：将 zip 包解压到 `SKILLS_DIR/`：
   ```bash
   # 先 cd 到 SKILLS_DIR，再解压（zip 内部已包含 <skill-name>/ 目录）
   cd "SKILLS_DIR" && unzip /tmp/<skill-name>.zip
   ```
6. **删除临时 zip 包**：
   ```bash
   rm -f /tmp/<skill-name>.zip
   ```
7. **验证**：确认 `SKILLS_DIR/<skill-name>/SKILL.md` 存在

**单个下载示例：**
```
用户：下载 sales-lead-tracker
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    正在下载 sales-lead-tracker.zip...
    ✅ 下载完成，MD5 校验通过。

    删除本地旧目录...
    解压到 /Users/dicsonpan/.trae-cn/skills/...
    清理临时文件...

    ✅ 下载成功！
    技能路径：/Users/dicsonpan/.trae-cn/skills/sales-lead-tracker/SKILL.md
    技能已就绪，可以使用。
```

**批量下载示例：**
```
用户：下载所有技能
AI：正在下载全部 2 个技能...

    ✅ sales-lead-tracker.zip → 解压完成
    ✅ student-growth-tracker.zip → 解压完成

    全部下载完成！
```

**实现步骤（批量下载）：**
1. 先调用 `GET /api/skills` 获取完整列表
2. 遍历每个技能（key 为 `xxx.zip`），提取技能名（去掉 `.zip`）
3. 对每个技能执行上述完整下载流程
4. 汇总报告下载结果

---

### 模块三：上传新技能（推送本地到服务器）

> **上传 = 将本地目录打包推送**：整个目录打 zip → 上传 zip → 下载回传校验 MD5 → 清理临时 zip

**调用 `POST /api/skills`（需要 ADMIN 权限）：**

```bash
# ⚠️ 必须将 SKILLS_DIR 替换为实际绝对路径！

# 第一步：打包 zip（在 SKILLS_DIR 下执行，排除 .credentials.json）
cd "SKILLS_DIR" && zip -r /tmp/<skill-name>.zip <skill-name>/ -x "*/.credentials.json"

# 第二步：上传 zip 包
curl -s -X POST https://api.sparkminds.io/api/skills \
  -H "Authorization: Bearer <token>" \
  -F "file=@/tmp/<skill-name>.zip;filename=<skill-name>.zip;type=application/zip"

# 第三步：清理临时 zip
rm -f /tmp/<skill-name>.zip
```

> **🔴 关键规则：**
> 1. `zip` 命令必须在 `SKILLS_DIR` 目录下执行，这样 zip 包内部结构为 `<skill-name>/...`
> 2. **必须排除 `.credentials.json`**：`-x "*/.credentials.json"`，防止泄露密码
> 3. 路径含空格时（如 DuMate），`cd` 和 `zip` 命令中的路径需用引号包裹

**上传流程：**
1. **确定路径**：按「运行环境检测」确定 `SKILLS_DIR` 的绝对路径
2. **确认本地技能目录存在**：用 LS 工具检查 `SKILLS_DIR/<skill-name>/` 存在，且包含 `SKILL.md`
3. **检查服务器是否已有同名技能**：调用 `GET /api/skills` 查看列表
4. **打包 zip**：在 `SKILLS_DIR` 下执行 `zip -r /tmp/<skill-name>.zip <skill-name>/ -x "*/.credentials.json"`
5. **上传**：
   - 若服务器**不存在**同名技能 → 使用 `POST /api/skills` 上传
   - 若服务器**已存在**同名技能 → 提示用户确认后，使用 `PUT /api/skills/<skill-name>.zip` 更新
6. **验证**：从服务器重新下载该 zip 包到 `/tmp/`，计算 MD5 与本地 `/tmp/<skill-name>.zip` 对比（或对比解压后内容），确认上传成功
7. **清理临时文件**：删除 `/tmp/<skill-name>.zip`

**返回格式：**
```json
{"success":true,"key":"<skill-name>.zip","message":"Skill added successfully"}
```

**交互示例（Trae CN 环境）：**
```
用户：上传 sparkminds-skills 技能
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    检查本地技能目录... 存在，包含 SKILL.md
    检查服务器是否已有同名技能... 不存在，使用上传模式。

    正在打包 zip（排除 .credentials.json）...
    正在上传 sparkminds-skills.zip...

    ✅ 上传成功！
    服务器文件名：sparkminds-skills.zip
    [验证] 从服务器下载回传，MD5 一致。
    [清理] 临时文件已删除。
```

**服务器已存在同名技能时的处理：**
```
用户：上传 sales-lead-tracker 技能
AI：⚠️ 服务器上已存在 sales-lead-tracker.zip
    如需覆盖，请使用"更新技能"功能。
    是否要更新覆盖？

用户：是
AI：正在打包并更新...
    ✅ 更新成功！
    [验证] MD5 一致。
```

---

### 模块四：更新已有技能（推送本地到服务器，覆盖）

> **更新到服务器 = 打包覆盖**：整个目录打 zip → PUT 覆盖 → 下载回传校验 MD5 → 清理临时 zip

**调用 `PUT /api/skills/{filename}`（需要 ADMIN 权限）：**

```bash
# ⚠️ 必须将 SKILLS_DIR 替换为实际绝对路径！

# 第一步：打包 zip
cd "SKILLS_DIR" && zip -r /tmp/<skill-name>.zip <skill-name>/ -x "*/.credentials.json"

# 第二步：PUT 更新
curl -s -X PUT https://api.sparkminds.io/api/skills/<skill-name>.zip \
  -H "Authorization: Bearer <token>" \
  -F "file=@/tmp/<skill-name>.zip;filename=<skill-name>.zip;type=application/zip"

# 第三步：清理临时 zip
rm -f /tmp/<skill-name>.zip
```

**更新流程：**
1. **确定路径**：按「运行环境检测」确定 `SKILLS_DIR` 的绝对路径
2. **确认本地技能目录存在**：用 LS 工具检查 `SKILLS_DIR/<skill-name>/` 存在，且包含 `SKILL.md`
3. **确认服务器上已有该技能**：若不存在，提示改用上传
4. **打包 zip**：在 `SKILLS_DIR` 下执行 `zip -r /tmp/<skill-name>.zip <skill-name>/ -x "*/.credentials.json"`
5. **PUT 更新**：`curl -X PUT .../api/skills/<skill-name>.zip`
6. **验证**：从服务器重新下载该 zip 包，计算 MD5 与本地对比，确认更新成功
7. **清理临时文件**：删除 `/tmp/<skill-name>.zip`

**交互示例（DuMate 环境）：**
```
用户：更新服务器上的 sales-lead-tracker 技能
AI：[检测运行环境：DuMate]
    SKILLS_DIR = /Users/dicsonpan/Library/Application Support/qianfan-desktop-app/qianfan_desk_xdg/cb18f6c82d304ab3860ba8248226a660/data/skills/user/

    正在打包 zip（排除 .credentials.json）...
    正在更新 sales-lead-tracker.zip...

    ✅ 更新成功！
    [验证] 从服务器下载回传，MD5 一致。
    [清理] 临时文件已删除。
```

**智能判断上传 vs 更新：**

当用户说"上传"、"同步"、"发布"时，自动执行以下逻辑：
1. 调用 `GET /api/skills` 检查服务器是否已有同名技能（`<skill-name>.zip`）
2. 若已有 → 自动使用 `PUT` 更新，并告知用户
3. 若没有 → 使用 `POST` 上传

---

### 模块五：删除技能

**调用 `DELETE /api/skills/{filename}`（需要 ADMIN 权限）：**

```bash
curl -s -X DELETE https://api.sparkminds.io/api/skills/<skill-name>.zip \
  -H "Authorization: Bearer <token>"
```

**交互示例：**
```
用户：删除服务器上的 test-skill 技能
AI：⚠️ 确认要删除服务器上的 test-skill.zip 吗？
    此操作不可恢复。

用户：确认
AI：正在删除...
    ✅ 技能 test-skill.zip 已从服务器删除。
```

**注意事项：**
- 删除操作必须向用户二次确认
- 仅删除服务器上的技能 zip 包，不影响本地文件
- 用户取消则中止操作

---

## 完整工作流

### 场景一：首次使用 + 下载技能

```
用户：下载技能 sales-lead-tracker
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    检测到您尚未配置 SparkMinds 账号，请提供登录凭证：
    用户名：？
    密码：？

用户：dicson，P@ssw0rd_api
AI：正在验证凭证...
    ✅ 登录成功！角色：ADMIN
    凭证已保存到本地。

    正在下载 sales-lead-tracker.zip...
    ✅ 下载完成，zip 完整性校验通过。

    删除本地旧目录...（若存在）
    解压到 /Users/dicsonpan/.trae-cn/skills/...
    清理临时 zip 文件...

    ✅ 下载成功！
    技能路径：/Users/dicsonpan/.trae-cn/skills/sales-lead-tracker/SKILL.md
```

### 场景二：上传本地技能到服务器

```
用户：把 competition-assistant 技能上传到服务器
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    正在读取凭证...
    ✅ 凭证已加载，正在登录...

    检查本地技能目录... 存在，包含 SKILL.md
    检查服务器是否已有同名技能... 不存在，使用上传模式。

    正在打包 zip（排除 .credentials.json）...
    正在上传 competition-assistant.zip...

    ✅ 上传成功！
    [验证] 从服务器下载回传，MD5 一致。
    [清理] 临时文件已删除。
```

### 场景三：本地修改后同步到服务器

```
用户：同步 sales-lead-tracker 到服务器
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    正在读取凭证...
    ✅ 凭证已加载，正在登录...

    服务器上已存在 sales-lead-tracker.zip，使用更新模式。

    正在打包 zip（排除 .credentials.json）...
    正在更新 sales-lead-tracker.zip...

    ✅ 更新成功！
    [验证] 从服务器下载回传，MD5 一致。
    [清理] 临时文件已删除。
```

### 场景四：批量下载所有技能

```
用户：下载所有技能
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    正在读取凭证...
    ✅ 凭证已加载，正在登录...

    获取技能列表... 共 2 个技能

    正在下载并解压...
    ✅ sales-lead-tracker.zip → 解压完成
    ✅ student-growth-tracker.zip → 解压完成

    全部下载完成！
```

### 场景五：从服务器更新本地技能

```
用户：从服务器更新本地的 student-growth-tracker
AI：[检测运行环境：Trae CN]
    SKILLS_DIR = /Users/dicsonpan/.trae-cn/skills/

    正在读取凭证...
    ✅ 凭证已加载，正在登录...

    正在下载 student-growth-tracker.zip...
    ✅ 下载完成，zip 完整性校验通过。

    ⚠️ 本地已存在 student-growth-tracker/ 目录，将被覆盖。
    确认继续？

用户：确认
AI：删除本地旧目录...
    解压到 /Users/dicsonpan/.trae-cn/skills/...
    清理临时 zip 文件...

    ✅ 更新成功！
    技能路径：/Users/dicsonpan/.trae-cn/skills/student-growth-tracker/SKILL.md
```

---

## 使用指南

### 快速参考

| 用户意图 | 操作方向 | 流程 | API 调用 |
|----------|----------|------|----------|
| 查看有哪些技能 | - | 列表 | `GET /api/skills` |
| 下载某个技能 | 服务器→本地 | 下载 zip → 校验 → 删旧目录 → 解压 → 删 zip | `GET /api/skills/{name}.zip` |
| 上传新技能 | 本地→服务器 | 打 zip → 上传 → 校验 → 删 zip | `POST /api/skills` |
| 更新服务器技能 | 本地→服务器 | 打 zip → PUT → 校验 → 删 zip | `PUT /api/skills/{name}.zip` |
| 更新本地技能 | 服务器→本地 | 同"下载" | `GET /api/skills/{name}.zip` |
| 删除技能 | - | 删除服务器 zip | `DELETE /api/skills/{name}.zip` |
| 修改登录账号 | - | 重新输入并保存 | - |

### 自然语言示例

```
用户：查看技能列表
用户：下载 sales-lead-tracker
用户：下载所有技能
用户：上传 sparkminds-skills
用户：更新服务器上的 student-growth-tracker
用户：同步 sales-lead-tracker 到服务器
用户：从服务器更新本地的 student-growth-tracker
用户：删除 test-skill
用户：更新凭证
```

---

## 注意事项

1. **🔴 zip 包格式**：服务器上技能以 `.zip` 格式存储，每个 zip 包含完整的技能目录（含 SKILL.md 及附属文件）。不再是单个 .md 文件
2. **🔴 排除凭证文件**：打包 zip 时**必须**排除 `.credentials.json`：`zip -r ... -x "*/.credentials.json"`，防止泄露密码
3. **🔴 下载即覆盖**：下载/更新本地技能时，会**删除整个本地旧目录**再解压。若本地有未保存的修改，应先提示用户确认
4. **🔴 上传/更新后必须验证**：上传或更新技能后，必须从服务器重新下载 zip 包，对比 MD5，确认服务器上的内容与本地一致
5. **🔴 路径自适应**：本技能支持多 App 运行环境，每次执行操作前必须按「运行环境检测」章节确定 `SKILLS_DIR` 的绝对路径，直接写入命令中，严禁使用未定义的变量名
6. **权限要求**：上传、更新、删除操作需要 `ADMIN` 角色；查看和下载仅需认证
7. **Token 时效**：每次操作前重新登录获取新 Token，避免 Token 过期问题
8. **凭证安全**：凭证文件存储在本地 `.credentials.json` 中，请勿提交到版本控制系统
9. **文件名规范**：服务器上技能文件以 `.zip` 结尾，本地技能目录名为去掉 `.zip` 的部分
10. **覆盖保护**：上传时若服务器已有同名技能，应提示用户使用"更新"而非直接上传
11. **删除确认**：删除操作不可恢复，必须向用户二次确认
12. **Cloudflare 防护**：服务器部署在 Cloudflare 上，必须使用 `curl` 发起请求
13. **临时文件**：所有临时 zip 文件使用 `/tmp/` 目录，操作完成后必须清理
14. **路径含空格**：DuMate 等环境的 `SKILLS_DIR` 可能含空格，命令中需用引号包裹路径

---

## 更新日志

### v2.0.0 (2026-07-01)
- **重大变更**：技能存储格式从单个 `.md` 文件改为 `.zip` 压缩包，支持完整目录结构（SKILL.md + 模板 + 附属文件）
- **变更**：上传流程改为：整个目录打 zip → 上传 zip → 下载回传校验 MD5 → 清理临时 zip
- **变更**：下载流程改为：下载 zip → MD5 校验 → 删除本地旧目录 → 解压 → 删 zip
- **新增**：打包时自动排除 `.credentials.json`，防止泄露密码
- **新增**：下载时若本地已存在旧目录，先提示用户确认再覆盖删除
- **新增**：使用 `/tmp/` 作为临时 zip 缓存目录，操作完成后自动清理
- **优化**：所有操作流程更严谨，增加完整性校验步骤

### v1.2.0 (2026-07-01)
- 修复上传/更新技能时路径变量未展开的问题
- 强制要求在命令中直接写入实际绝对路径
- 新增上传/更新后 MD5 验证

### v1.1.0 (2026-07-01)
- 新增运行环境自动检测机制，支持 Trae CN、DuMate、Cursor 等多 App

### v1.0.0 (2026-07-01)
- 初始版本

---

**创建时间：** 2026-07-01
**版本：** 2.0.0
**作者：** 小创 · 创智教育 AI 助理
