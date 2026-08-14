# DeepSeek Harness (dsh) + ModLens 识图插件安装配置指南

> 目标：让 **DeepSeek/GLM 等纯文本模型** 拥有看图能力。原理：ModLens 把图片转发给外部**视觉引擎**（本指南用 opencode-go 订阅里的 MiMo-V2.5），转成结构化文本证据（OCR / 版面 / 实体 / 主色），再交给 DeepSeek 回答。
>
> 本指南基于 Windows + Git Bash 实测（2026-08），macOS / Linux 命令基本一致（路径差异已标注）。

---

## 0. 总体架构

```
浏览器 (dsh web UI)
   │ 粘贴图片 / 模型调用 read_image
   ▼
dsh (deepseek-harness, cordis 插件系统)
   │ web profile = 一组 bundle 层叠 + cordis.patch.yml 覆盖层
   ▼
ModLens 插件 (@liustack/modlens)
   │ 注册 read_image 工具 + "(modlens vision)" 模型条目
   │ 包装上游 = opencode-go 路由（本次配置的关键）
   ▼
dsh 聊天路由：opencode-go provider → deepseek-v4-flash（走你的订阅）
dsh 识图路由：modlens → ~/.modlens/config.json → opencode-go API → mimo-v2.5
   ▼
结构化 JSON 证据 (summary / ocr / layout / semantics / visual)
   ▼
DeepSeek 纯文本模型 基于证据回答
```

**关键点 1：图片从不直接发给 DeepSeek**（它没有视觉能力），先被外部引擎转成文本证据。
**关键点 2：有两条独立的 opencode-go 链路**——聊天走 dsh 的 `opencode-go` provider 路由，识图走 modlens 配置的视觉引擎（同一个订阅，两个入口）。

---

## 1. 前置条件

| 组件 | 要求 | 本机实测 |
|------|------|----------|
| Node.js | ≥ 22.19（dsh 要求 `^22.19.0 \|\| >=24.0.0`）；modlens 要求 ≥ 22.13 | v24.18.0 ✅ |
| pnpm | dsh 项目固定 `pnpm@11.7.0`（`packageManager` 字段） | 11.7.0 ✅ |
| dsh | 已能 `dsh web` 正常跑 | deepseek-harness 本地构建 ✅ |
| 视觉引擎 | 至少一个：Antigravity CLI / Gemini key / OpenAI 兼容端点 / 本机 opencode 登录态 | opencode-go 订阅 ✅ |

---

## 2. dsh 本体安装（新机器首次）

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness

# 安装 pnpm（二选一；corepack 在 Windows 常因权限失败，推荐 npm 全局装）
# corepack enable        # 需管理员权限写 C:\Program Files\nodejs
npm i -g pnpm@11.7.0     # 与项目 packageManager 版本一致

pnpm install
pnpm run build           # 构建 lib + web（约 1-2 分钟）
pnpm dsh web             # 启动，默认 http://127.0.0.1:3080
```

> 后台运行（Git Bash）：`nohup pnpm dsh web > /tmp/dsh-web.log 2>&1 &`
> 停止：`netstat -ano | grep :3080` 拿 PID → `taskkill //F //PID <pid>`

### 2.1 封装成命令（可选）

把启动/停止封装为 `dsh-web` / `dsh-web-stop`（脚本放 `~/bin/`，已在 Windows 用户 PATH）：

- `~/bin/dsh-web`（bash 脚本）：cd 进仓库 → 检测端口占用（幂等）→ `nohup pnpm dsh web` 后台启动 → 轮询就绪 → 打印 URL
- `~/bin/dsh-web-stop`（bash 脚本）：按端口 3080 找到 PID → `taskkill` → 确认释放
- `~/bin/dsh-web.cmd` / `~/bin/dsh-web-stop.cmd`：PowerShell/CMD 入口，转发给 Git Bash 的 bash 执行（`.cmd` 注释必须纯 ASCII，中文注释会被 cmd 按 GBK 误解析报错）

脚本头部 `DSH_DIR` 一行是仓库路径，换机器改这一行即可。

---

## 3. 安装 ModLens 插件（含"发布冷静期"坑）

### 3.1 安装命令

```sh
cd deepseek-harness
pnpm dsh plugin --profile web add @liustack/modlens@latest
```

### 3.2 ⚠️ 必踩的坑：pnpm 发布冷静期

pnpm ≥ 10/11 默认开启 `minimumReleaseAge`（**10 天窗口**）：解析**范围/`@latest` 标签**时，会静默回退到冷静期外的**旧版本**。

ModLens 的 `dsh.bundle` 声明 **3.9.0 起才有**。回退到 3.5.0 这类旧版后，dsh 会报：

```
dsh: warning: @liustack/modlens declares no dsh.bundle — installed as a plain dependency, not a profile layer
```

插件装上了但完全不生效（工具不出来）。

**修复（两步都要做）：**

① 持久排除（写进 profile 的 workspace 配置，未来新版本即时可用）：

```yaml
# 文件：~/.dsh/profiles/web/pnpm-workspace.yaml
minimumReleaseAgeExclude:
  - '@liustack/modlens'
```

> 注意：该项必须放在 `minimumReleaseAgeExclude:` 列表下，别追加到文件末尾（可能落进 `allowBuilds` 块导致解析错误）。

② 显式指定版本重装（显式版本号跳过冷静期，见 pnpm#9989）：

```sh
pnpm dsh plugin --profile web add @liustack/modlens@3.12.1   # 版本号按 npm 最新
```

### 3.3 验证挂载

```sh
pnpm dsh plugin --profile web list
# 应看到：@liustack/modlens@3.12.1

cat ~/.dsh/profiles/web/package.json
# "dsh": { "profile": { "bundles": [ ..., "@liustack/modlens" ] } }
```

modlens 是 **host 端 bundle**（通过包内 `cordis.patch.yml` 挂载，无 client 入口），所以浏览器 boot 配置里看不到它是正常的；用 `pnpm dsh --profile web --dump-config | grep modlens` 确认 patch 节点已进配置树。

---

## 4. 配置视觉引擎：opencode-go 订阅 + MiMo-V2.5

> 本节配置的是 **modlens 的识图引擎**（`~/.modlens/config.json`），与 dsh 侧模型路由（第 5 节）相互独立。

### 4.1 获取你的 opencode 信息（一次性）

**① API Key**（opencode 登录后自动保存）：

- Windows: `%APPDATA%\opencode\auth.json`（即 `~/AppData/Roaming/opencode/auth.json`）
- Linux/macOS: `~/.local/share/opencode/auth.json`

```json
{ "opencode-go": { "type": "api", "key": "sk-xxxx..." } }
```

**② OpenAI 兼容端点**（在 opencode 配置里查，通常 `~/.config/opencode/opencode.json` 或 `%APPDATA%\opencode\opencode.json`）：

```json
{
  "provider": {
    "opencode-api": {
      "name": "OpenCode Go",
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "apiKey": "sk-xxxx...",
        "baseURL": "https://opencode.ai/zen/go/v1"
      }
    }
  }
}
```

**③ 确认模型与视觉能力**：

```sh
opencode models | grep mimo
# opencode-go/mimo-v2.5      ← 视觉验证通过（本机实测）
# opencode-go/mimo-v2.5-pro  ← 更强，同样可用
```

### 4.2 先做视觉能力实测（可选但强烈建议）

直接向 opencode-go 端点发一个带图片的请求，确认模型真能看图（避免装完才发现不行）：

```sh
curl -s https://opencode.ai/zen/go/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <你的opencode-go key>" \
  -d '{
    "model": "mimo-v2.5",
    "messages": [{"role":"user","content":[
      {"type":"text","text":"这张图是什么颜色？"},
      {"type":"image_url","image_url":{"url":"data:image/png;base64,<图片base64>"}}
    ]}]
  }'
```

成功返回 `HTTP 200` 且内容提到颜色/内容 = 视觉可用。（不支持图片的模型，opencode 网关会报 `model does not support image input`。）

### 4.3 配置 modlens（核心配置）

modlens CLI 装在 dsh profile 里，路径：`~/.dsh/profiles/web/node_modules/.bin/modlens`

```sh
cd ~/.dsh/profiles/web
M=node_modules/.bin/modlens

# ① 指向 opencode-go 的 OpenAI 兼容端点
$M config set openai.baseUrl https://opencode.ai/zen/go/v1
# ② 填入 opencode-go 的 key
$M config set openai.apiKey <你的opencode-go key>
# ③ 指定视觉模型（视觉版）
$M config set openai.model mimo-v2.5
# ④ 设为默认 provider
$M config set provider openai
```

**配置落盘位置：`~/.modlens/config.json`**

```json
{
  "provider": "openai",
  "providers": {
    "openai": {
      "apiKey": "sk-xxxx...",
      "baseUrl": "https://opencode.ai/zen/go/v1",
      "model": "mimo-v2.5"
    }
  }
}
```

> 配置优先级：CLI 参数 > 环境变量 > 配置文件 > 内置默认。
> 环境变量形式：`OPENAI_API_KEY` / `OPENAI_BASE_URL` 可覆盖文件。

### 4.4 端到端验证

```sh
# 体检（不耗额度、不发网络请求）
$M doctor
# 应看到：openai: baseUrl: file, apiKey: file, model: file [ok]
#        Selected provider: openai

# 真实识别一张图（耗一次额度，约 5-10 秒）
$M analyze -i /path/to/image.png
# 应返回结构化 JSON：summary / ocr / layout / semantics / visual（含 dominant_colors 等）
```

---

## 5. 让 dsh 聊天走 opencode-go（供应商配置）

> 本节是让 dsh 的 **DeepSeek 聊天模型本身**走 opencode-go 订阅（而不是官方 API）。可选——想直接用官方 deepseek 可以跳过，但本指南场景（用 opencode-go 订阅）建议配置。

### 5.1 在 web UI 添加 opencode-go 供应商

**设置 → 模型** 页面添加 provider：

- 路由名：`opencode-go`（dsh 的 pi-ai 目录**内置认识**这个路由，模型目录自动带上 18 个模型：deepseek-v4-flash/pro、glm、mimo-v2.5、qwen、kimi 等）
- API：`openai-completions`
- Base URL：`https://opencode.ai/zen/go/v1`
- 凭据：存你的 opencode-go key（走 dsh 的 credentials 服务）

落盘位置 `~/.dsh/settings.yaml`，形如：

```yaml
agent-default-model:
  provider: opencode-go
  model: deepseek-v4-flash
  reasoningEffort: max
llm-pi-ai:
  providers:
    opencode-go: {}     # 空对象 = 用 pi-ai 内置目录，无需手写 baseURL/models
```

> 若 `providers.opencode-go` 显示 `{}` 空对象，**不是没配置**——pi-ai 内置目录已提供端点与模型目录，空对象即"全部用内置默认"。模型路由经 UI 保存后即 ACTIVE。

### 5.2 让 modlens 包装 opencode-go 的模型（本次关键更新）

modlens 的 `registerVisionProvider` 只包装 `config.upstream` 指定的一个 provider（默认 `deepseek-official`）。要让 **opencode-go 的 deepseek/glm 模型**也有识图包装，把 upstream 指过去：

```yaml
# 文件：~/.dsh/profiles/web/cordis.patch.yml
- id: modlens
  config:
    upstream: opencode-go
```

改完**重启 dsh web**（`dsh-web-stop && dsh-web`）。

**效果**（重启后 API 实测 `llm.models`）：

| 模型组 | 内容 |
|--------|------|
| deepseek-official | DeepSeek-V4-Flash / DeepSeek-V4-Pro（原始，无包装） |
| **modlens vision** | **DeepSeek V4 Flash (modlens vision)** ✅ / **DeepSeek V4 Pro (modlens vision)** ✅ / GLM-5.1 / GLM-5.2 |
| opencode-go | 原始目录 18 个模型 |

包装条目路由：识图 → modlens → mimo-v2.5；聊天 → opencode-go 的 deepseek-v4-flash。**整个链路走你的订阅。**

> ⚠️ modlens 只支持一个 upstream——官方 deepseek-official 的识图包装被 opencode-go 取代。若需保留官方包装，可注册第二个 modlens 实例（`insert` 一个 `id: modlens-official`、`config.upstream: deepseek-official`、`config.providerId: deepseek-modlens-official`），但模型选择器会出现两套同名 "(modlens vision)" 条目，不易区分，不建议。

---

## 6. dsh 侧启用与使用

1. **重启 dsh web**（改了 profile 必须重启）：
   ```sh
   dsh-web-stop && dsh-web
   ```
2. **浏览器硬刷新**（Ctrl+Shift+R）—— 保证客户端资源重新加载。
3. 在模型选择器切到 modlens 提供的条目（来自 opencode-go 的包装，注意名字带空格是 opencode-go 目录风格）：
   - **`DeepSeek V4 Flash (modlens vision)`**（推荐，聊天走 opencode-go）
   - `DeepSeek V4 Pro (modlens vision)`
   - `GLM-5.1 (modlens vision)` / `GLM-5.2 (modlens vision)`
4. 直接**粘贴图片**到输入框（无需先存文件），模型会用 `read_image` 工具走 modlens → mimo-v2.5 识图后回答。

> 包装只覆盖 deepseek/glm 家族文本模型；有原生视觉的模型（mimo 等）不会被包装。

---

## 7. 可选优化与替代方案

### 7.1 关闭 thinking 提速

MiMo 是推理模型，识图时会花大量 reasoning token 重新推导转录内容。configure.md 建议：

```sh
$M config set openai.extraBody '{"thinking":{"type":"disabled"}}'
```

> 注意：opencode 网关是否接受该字段**未验证**，需实测；若 400 报错则清掉：
> `$M config set openai.extraBody ''`

### 7.2 换更强视觉模型

```sh
$M config set openai.model mimo-v2.5-pro   # 更强；识别更慢
```

### 7.3 其他视觉引擎（modlens 内置 provider）

| Provider | 命令 | 说明 |
|----------|------|------|
| gemini-api | `modlens config set gemini-api.apiKey <key>` + `config set provider gemini-api` | 免费 key（aistudio.google.com），5-10s，不过期 |
| antigravity-cli | 安装 `curl -fsSL https://antigravity.google/cli/install.sh \| bash` | 默认 provider，免 key，免费 |
| anthropic | `config set anthropic.apiKey <key>` | Claude API |
| openai (任意 OpenAI 兼容端点) | 见 4.3 | 通吃：DashScope/vLLM/各类网关 |
| 复用本机登录态 | `config set reuse.opencode true` 等 | 复用本机 opencode/codex/claude/pi/grok 登录，花对应账户额度 |

> modlens 的 failover 链：多个 provider 都配了的话，按固定顺序依次尝试（inline API 5-10s → 本地 agent），`config set provider <name>` 把某 provider 提到最前。

---

## 8. 其他机器快速部署清单

```sh
# ===== 1. dsh 本体 =====
git clone https://github.com/deepseek-ai/deepseek-harness.git && cd deepseek-harness
npm i -g pnpm@11.7.0
pnpm install && pnpm run build

# ===== 2. 装 modlens（绕冷静期两步）=====
echo "minimumReleaseAgeExclude:" >> ~/.dsh/profiles/web/pnpm-workspace.yaml   # 若文件已有该键则并入
echo "  - '@liustack/modlens'" >> ~/.dsh/profiles/web/pnpm-workspace.yaml
pnpm dsh plugin --profile web add @liustack/modlens@3.12.1   # 显式版本号

# ===== 3. 配 opencode-go 视觉引擎 =====
# 前提：本机 opencode 已登录 opencode-go（auth.json 有 key）
cd ~/.dsh/profiles/web
M=node_modules/.bin/modlens
$M config set openai.baseUrl https://opencode.ai/zen/go/v1
$M config set openai.apiKey <opencode-go key>      # 见 ~/AppData/Roaming/opencode/auth.json
$M config set openai.model mimo-v2.5
$M config set provider openai
$M doctor                                            # 体检

# ===== 4. dsh 聊天路由走 opencode-go + 包装 =====
# 4a. web UI 设置→模型 添加 opencode-go 供应商（baseURL/key），并把默认模型切到它
# 4b. 让 modlens 包装 opencode-go：
cat > ~/.dsh/profiles/web/cordis.patch.yml <<'EOF'
- id: modlens
  config:
    upstream: opencode-go
EOF

# ===== 5. 启动 =====
cd deepseek-harness && pnpm dsh web
# 浏览器硬刷新 → 模型切到 "DeepSeek V4 Flash (modlens vision)" → 粘贴图片
```

---

## 9. 故障排查速查

| 症状 | 原因 | 解决 |
|------|------|------|
| `declares no dsh.bundle — installed as a plain dependency` | pnpm 发布冷静期压到旧版 | 加 `minimumReleaseAgeExclude` + 显式版本重装（见 3.2） |
| pnpm 报 `Ignored build scripts` | pnpm 11 默认拦截 postinstall | 在 `~/.dsh/profiles/web` 下 `pnpm approve-builds --all` |
| `dsh: pnpm not found on PATH` | 未装 pnpm / 新终端未刷新 PATH | `npm i -g pnpm@11.7.0` 后重开终端 |
| 装了但工具不出现 | 装了旧版 / 没重启 / 没硬刷新 | `plugin list` 看版本 ≥3.9.0；重启 dsh web + Ctrl+Shift+R |
| 模型选择器没有 opencode-go 的 "(modlens vision)" 条目 | modlens 的 upstream 仍是默认 deepseek-official | 改 `cordis.patch.yml` 的 `upstream: opencode-go` 并重启 |
| 模型选择器里 opencode-go 无模型可选 | opencode-go 供应商没在 UI 保存成功 / 空配置未被目录识别 | 重新在 设置→模型 添加并保存；确认 `settings.yaml` 的 `llm-pi-ai.providers.opencode-go` 存在 |
| 识图报错 / 超时 | 网关不支持某字段 / 网络代理 | 检查 extraBody 字段；modlens 默认不走系统代理，需在请求侧配代理 |
| 粘贴图片没反应 | 模型选择器没切到 modlens vision 条目 | 切到 `DeepSeek V4 Flash (modlens vision)`（opencode-go 组） |

---

## 10. 参考链接

- dsh 仓库: https://github.com/deepseek-ai/deepseek-harness
- ModLens 仓库: https://github.com/liustack/modlens （README.zh-CN.md / INSTALL.md / docs/troubleshooting.md / skills/modlens/references/configure.md）
- ModLens npm: https://www.npmjs.com/package/@liustack/modlens
- pnpm 发布冷静期 issue: https://github.com/pnpm/pnpm/issues/9989
- opencode: https://opencode.ai （订阅计划 Opencode Go）
- 小米 MiMo: https://github.com/XiaomiMiMo
