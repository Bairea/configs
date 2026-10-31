# DeepSeek Harness (dsh) 配置：模型供应商 + 插件

dsh 的模型供应商由 `@deepseek-ai/dsh-llm-pi-ai` 插件提供，配置写在 profile 的 patch 层
`~/.dsh/profiles/<profile>/cordis.patch.yml`（顶层是 loader patch 数组，按插件 `id` 覆盖配置）。

> **先确认 profile**：本文的 `<profile>` 指实际启动时用的那个 profile。
> 桌面版（`DeepSeek Harness.exe`）用的是 **`desktop`**，即 `~/.dsh/profiles/desktop/`；
> `web` 是 `dsh web` 命令行启动时用的另一个独立 profile（本机已存在但基本是空的）。
> 写到错的 profile 里不会有任何报错，只是"改了没反应"——这是最常见的踩坑。
> 当前 profile 可用环境变量 `$env:DSH_PROFILE` / `$env:DSH_PROFILE_DIR` 确认。

- `apiKeyEnv` 只写凭据引用名，密钥本身存 `~/.dsh/.credentials.yaml`。
- 手工声明的路由必须给出 `api`、`baseURL` 与非空的 `models` 列表。
- `reasoningEfforts` 的键是选择器里显示的等级，值是该等级在协议中实际发送的拼写。
- `retryPolicy` 由 `dsh-llm-retry` 执行，省略时默认 normal 模式、最多重试 5 次。

```yaml
# ~/.dsh/profiles/desktop/cordis.patch.yml
- id: llm-pi-ai
  name: "@deepseek-ai/dsh-llm-pi-ai"
  config:
    providers:
      zy-api-a:
        apiKeyEnv: ZY_API_A_API_KEY
        api: openai-completions
        baseURL: https://image.yydsapi.uno
        compat:
          thinkingFormat: openai
          supportsReasoningEffort: true
        models:
          - id: grok-4.6
            name: Grok 4.6
          - id: grok-4.7
            name: Grok 4.7
            contextWindow: 500000
            input:
              - text
              - image
            reasoningEfforts:
              low: low
              medium: medium
              high: high
              xhigh: xhigh
        retryPolicy:
          mode: normal
          maxRetries: 3
          retryableCodes:
            - EMPTY_RESPONSE
            - RATE_LIMIT
            - SERVER
            - TIMEOUT
            - TRANSPORT
            - INVALID_REQUEST
          backoff:
            initialDelayMs: 500
            maxDelayMs: 10000
            jitterRatio: 0.1
      zy-api-o1:
        apiKeyEnv: ZY_API_O1_API_KEY
        api: openai-completions
        baseURL: https://image.yydsapi.uno
        models:
          - id: gpt-5.6-sol
            name: GPT-5.6 Sol
          - id: gpt-6-astra
            name: GPT-6 Astra
          - id: gpt-6-sol
            name: GPT-6 Sol
          - id: gpt-6.1-sol
            name: GPT-6.1 Sol
        retryPolicy:
          mode: normal
          maxRetries: 3
          retryableCodes:
            - EMPTY_RESPONSE
            - RATE_LIMIT
            - SERVER
            - TIMEOUT
            - TRANSPORT
            - INVALID_REQUEST
          backoff:
            initialDelayMs: 500
            maxDelayMs: 10000
            jitterRatio: 0.1
      open-design:
        displayName: Open Design (Responses)
        apiKeyEnv: OPEN_DESIGN_API_KEY
        api: openai-responses
        baseURL: https://amr-link.open-design.ai/v1
        models:
          - id: deepseek-v4.1-flash
            name: deepseek-v4.1-flash
            input:
              - text
              - image
            contextWindow: 1048000
            maxTokens: 384000
            reasoningEfforts:
              off: none
              minimal: minimal
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
      open-design-chat:
        displayName: Open Design (Chat)
        apiKeyEnv: OPEN_DESIGN_API_KEY
        api: openai-completions
        baseURL: https://amr-link.open-design.ai/v1
        models:
          - id: deepseek-v4.1-flash
            name: deepseek-v4.1-flash
            input:
              - text
              - image
            contextWindow: 1048000
            maxTokens: 384000
            reasoningEfforts:
              off: "off"
              minimal: minimal
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
      command-code:
        apiKeyEnv: COMMAND_CODE_API_KEY
        api: openai-completions
        baseURL: https://api.commandcode.ai/provider/v1
        models:
          - id: deepseek/deepseek-v4.1-flash
            name: DeepSeek V4.1 Flash (latest)
            contextWindow: 1000000
            maxTokens: 384000
            input:
              - text
              - image
            reasoningEfforts:
              off: "off"
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
          - id: deepseek/deepseek-v4.1-flash-fast
            name: DeepSeek V4.1 Flash Fast (latest)
            contextWindow: 1000000
            maxTokens: 384000
            input:
              - text
              - image
            reasoningEfforts:
              off: "off"
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
      command-code-response:
        displayName: Command Code (Responses)
        apiKeyEnv: COMMAND_CODE_API_KEY
        api: openai-responses
        baseURL: https://api.commandcode.ai/provider/v1
        models:
          - id: deepseek/deepseek-v4.1-flash
            name: DeepSeek V4.1 Flash (Responses)
            contextWindow: 1000000
            maxTokens: 384000
            input:
              - text
              - image
            reasoningEfforts:
              off: none
              minimal: minimal
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
          - id: deepseek/deepseek-v4.1-flash-fast
            name: DeepSeek V4.1 Flash Fast (Responses)
            contextWindow: 1000000
            maxTokens: 384000
            input:
              - text
              - image
            reasoningEfforts:
              off: none
              minimal: minimal
              low: low
              medium: medium
              high: high
              xhigh: xhigh
              max: max
```

## 凭据

配置里引用的四个凭据名需要在 `~/.dsh/.credentials.yaml` 中存在，否则请求会以 `MISSING_CREDENTIAL` 失败。

| 凭据名 | 对应供应商 |
|--------|-----------|
| `ZY_API_A_API_KEY` | `zy-api-a` |
| `ZY_API_O1_API_KEY` | `zy-api-o1` |
| `OPEN_DESIGN_API_KEY` | `open-design`、`open-design-chat` |
| `COMMAND_CODE_API_KEY` | `command-code`、`command-code-response` |

## 生效

改完 `cordis.patch.yml` 需要重启 dsh（桌面版退出再开，命令行重启 `dsh web`），再在界面里硬刷新
（`Ctrl+Shift+R`），模型选择器才会出现新路由。

## 插件

本机实际装在这一个 profile 里（`~/.dsh/profiles/desktop/`），宿主版本
`@deepseek-ai/dsh-desktop-runtime 0.2.1-rc.2`。本机是 macOS，`@copylee/dsh-computer-use`
声明 `"os": ["win32"]`，无法安装/激活，未装；其余三个跨平台插件 + DSH-Notify-Me 已装：

| 插件 | 版本 | 行 id（写 `cordis.patch.yml` 时用） | 作用 |
|------|------|-----------------------------------|------|
| `dsh-web-tools` | 0.3.8（GitHub 源，文档初版写 0.3.7） | `dsh-web-tools` | 多家搜索/抓取供应商接入 `web_search` / `web_fetch` |
| `@changfenhuang/dsh-genui` | 0.14.0 | `genui` | 把 `dsh-ui` 围栏渲染成可交互组件 |
| `dsh-better-sidebar` | 0.28.0 | `better-sidebar` | 右侧栏：文件树 / 编辑器 / Git / 任务 / 底部工作台 |
| `dsh-notify-me` | 1.17.0（`github:chromoany/dsh-notify-me`） | `dsh-notify-me` | 系统通知 / 提示音 / 标题标记，后台任务审批与完成提醒 |
| `@copylee/dsh-computer-use` | 0.8.0（**未装**：仅 Windows） | `copylee-computer-use` | 看屏幕、点鼠标、敲键盘、开关应用 |

版本下限（装之前对一下宿主版本，对不上插件会静默不激活）：

- `@copylee/dsh-computer-use`：DSH ≥ 0.2.0-rc.2，且**只有 Windows**（`"os": ["win32"]`）——本机 macOS 未装
- `dsh-web-tools`：DSH ≥ 0.1.0-rc.6
- `@changfenhuang/dsh-genui`：DSH `^0.2.0-rc.1 || ^0.2.1-rc.1`
- `dsh-better-sidebar`：DSH **≥ 0.2.1-rc.1**；0.2.0-rc.2 上只能钉 `0.26.1`，0.2.0-rc.1 用 `0.24.1`
- `dsh-notify-me`：实测 1.17.0 在 0.2.1-rc.2 上正常；DSH Hub 标注兼容范围含 `=0.2.0-rc.1 <0.3.0-0`

### 安装

桌面版走 **设置 → 插件 → 添加插件**，填包名 + 版本即可。命令行：本机 `dsh` 不在 PATH 里，
但 macOS 上 `DeepSeek Harness.app` 自带 CLI，直接用
`/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh` 即可（无需 `npm i -g`）：

```bash
DSH="/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh"
# 本机 macOS：computer-use 仅 Windows，跳过；其余跨平台插件 + Notify-Me 已装
"$DSH" plugin --profile desktop add @changfenhuang/dsh-genui@0.14.0
"$DSH" plugin --profile desktop add dsh-better-sidebar@0.28.0
"$DSH" plugin --profile desktop add github:A3Boy/dsh-web-tools
"$DSH" plugin --profile desktop add github:chromoany/dsh-notify-me
# 若要在 Windows 上补装电脑使用插件：
# "$DSH" plugin --profile desktop add @copylee/dsh-computer-use@0.8.0
```

装完以后 `package.json` 会自动多出两处，**不要手改**：

```json
{
  "dependencies": {
    "@changfenhuang/dsh-genui": "0.14.0",
    "dsh-better-sidebar": "0.28.0",
    "dsh-web-tools": "github:A3Boy/dsh-web-tools",
    "dsh-notify-me": "github:chromoany/dsh-notify-me"
  },
  "dsh": { "profile": { "bundles": [ "@changfenhuang/dsh-genui", "dsh-better-sidebar", "dsh-web-tools", "dsh-notify-me" ] } }
}
```

每个 bundle 自带的 `cordis.patch.yml` 负责 `insert` 出插件行（`copylee-computer-use` / `genui` /
`better-sidebar` / `dsh-web-tools`），所以正常情况下 profile 自己的 `cordis.patch.yml` 里**不要**再写
`insert`。只有要改配置时才写一条同 `id` 的覆盖项。

### 1. 电脑使用：`@copylee/dsh-computer-use`

装完直接生效：会话里多出 `computer`、`computer_batch`、`open_application`、`windows`、
`ui_elements`、`clipboard`、`switch_display`、`request_access`、`app_skill` 等工具，
系统提示里也会多一段"Computer use (Windows desktop)"。

配置项（`id: copylee-computer-use`，全部有默认值，一般不用写）：

```yaml
# ~/.dsh/profiles/desktop/cordis.patch.yml
- id: copylee-computer-use
  name: "@copylee/dsh-computer-use"
  config:
    accessMode: per-app       # per-app（按应用授权）| allow-all（不弹审批）
    overlay: true             # 橙色光晕 + 顶部提示条（不进截图）
    hostWindow: pet           # pet 迷你进度卡片 | card 悬浮卡片 | minimize | keep
    autoScreenshot: true      # 每个动作后自动回传截图
    settleMs: 400             # 动作后至少等这么久，画面静止即截图
    maxLongEdge: 1366         # 截图最长边，越大越费 token
    maxPixels: 1150000
    jpegQuality: 70
    typingMode: stream        # stream 分段粘贴 | type 逐字 | paste 一次粘贴
    blockedApps: []           # 例如 ["alipay"]；常见密码管理器默认已禁
    nativeHelper: true        # 用 DSH 自带 koffi 在进程内截图/输入
```

踩坑：

- **别装第二个 computer-use 插件/skill**。这类插件都抢同一批工具名（`computer`、`screenshot`…），
  同时装会互相顶掉甚至加载失败；之前装过 `dsh-computer-use-win`、Hermes 之类的一定先停用。
- **审批模式决定体感**。默认 `per-app`，模型每次要 `request_access` 列应用让你批准。会话权限是
  "完全权限 / 自动审查"（不弹审批）时这一步自动通过——本机就是这种状态，所以它看起来"直接就能动"。
  只想让它碰固定几个应用，就留 `per-app` 并在设置里配。
- **UAC 弹窗、锁屏、以管理员身份运行的窗口点不动**（Windows UIPI 限制），这不是插件 bug，
  遇到时它会请你手动处理。
- `helper` 首次用 Win 自带的 `csc.exe` 编译（约 1 秒，缓存在 `%LOCALAPPDATA%\dsh-computer-use\`）。
  开了"智能应用控制（Smart App Control）"的机器 `csc` 可能被拦，插件会自动退回在 PowerShell 里跑 helper；
  再不行就关掉 `nativeHelper` 全部交给 helper。
- 截图会作为图片附件进会话历史，**删了仍被引用的截图会让那个会话打不开**；插件自己不会动它们，
  `~/.dsh/attachments` 里的孤儿截图由 DSH 定期清理。
- `app_skill` 的笔记写在 `~/.dsh/computer-use-skills/`，内置 Word/Excel/PPT/Chrome/微信等十几份技能；
  你自己追加的部分单独存，插件升级不会覆盖。

### 2. 联网：`dsh-web-tools`

**npm 上没有这个包**（`registry.npmjs.org/dsh-web-tools` → 404），只能从 GitHub 装：
`github:A3Boy/dsh-web-tools`。tarball 里带了编译好的 `lib/`，所以不需要构建，但**要能连 GitHub**。

它替换的是 DSH 自带的 `web` / `tool-web` 两行，把 `web_search` / `web_fetch` 接到 Exa、Tavily、
Firecrawl、Parallel、Brave、You.com、Jina、SearXNG 以及小红书、X/Twitter 上。

配置走 **设置 → 联网搜索**（供应商、API Key、路由策略、回退链都在那儿，键存在 DSH 的 credentials 里）。

踩坑：

- **不要在 `cordis.patch.yml` 里自己覆盖 `tool-web`**。patch 的 `config` 是整块替换、不深合并，
  少写一个键就回落默认值；尤其 `fetch: true` 决定 `web_fetch` 工具本身有没有注册，
  漏了它模型就完全看不到 `web_fetch`。非要改就照抄整份：

  ```yaml
  - id: tool-web
    disabled: false
    config:
      search: true
      fetch: true
      searchMaxQueries: 4
      searchMaxResults: 8
      searchTimeoutMs: 60000
      fetchTimeoutMs: 30000
      fetchMaxOutputChars: 200000
  ```

- **SearXNG 要开 JSON 输出**，否则报 `no usable provider` / 403：`settings.yml` 里
  `search.formats` 加 `- json` 再重启容器。
- 小红书 / X 用的是独立浏览器 profile（CDP 直连本地 Edge/Chrome），**不需要装扩展、不需要 Playwright**，
  但要求那个浏览器里已经登录；登录态失效会明确报 `login wall` / `search-restricted`，
  不会拿索引内容冒充"原生结果"。
- 升级命令 `dsh plugin --profile desktop update dsh-web-tools`；从本地路径/软链接装过的话，
  缓存问题要在 profile 目录 `pnpm install` 一次。

### 3. 生成式 UI：`@changfenhuang/dsh-genui`

包名必须是 **`@changfenhuang/dsh-genui`**。npm 上还有一个无 scope 的 `dsh-genui`，
那是别人写的 Vue/OpenTiny 实现，装了会因为 peer 依赖不匹配直接失败。

装完会多出：

- `render_ui`、`validate_dsh_ui` 两个工具
- 一个 `genui` 技能（模型靠它知道 `dsh-ui` 有哪些组件、字段怎么写）
- 浏览器端把 `dsh-ui` 围栏渲染成 stat / table / chart / mermaid / plot / quiz / panel 等组件

配置（`id: genui`）：

```yaml
- id: genui
  name: "@changfenhuang/dsh-genui"
  config:
    renderUiTool: true    # 关掉就不注册 render_ui 工具，围栏和 panel 照常工作
    fenceFeedback: true   # 围栏 JSON 校验失败时，同一轮把错误回给模型让它自己改
```

踩坑：

- **从老组织名迁过来要清干净**：v0.9.2 之前是 `@omdsh-dev/dsh-genui`，
  先 `dsh plugin --profile desktop remove @omdsh-dev/dsh-genui` 再装新的，否则两个一起挂。
- **别用 `link:` 装刚 clone 的目录**。`link:` 不会打包/解析 peer 资源（mermaid、three.js、React），
  会缺件；`link:` 只用于本地开发。
- 围栏语言必须严格是 `dsh-ui`，body 必须是合法 JSON。渲染不出来先看控制台有没有
  `[genui] client active; fence-channel=dom`：没有＝客户端没加载（刷新/重装），
  有＝围栏本身写错了（用 `validate_dsh_ui` 先验一遍）。
- ECharts / Mermaid / 3D 这几个引擎是首次用到时才懒加载的，第一次会有一两秒空白；
  换过版本或清过缓存后硬刷新一次。
- 安装时 pnpm 11 的 `minimumReleaseAge` 会拦 24 小时内发布的新版本，本机安装时
  `pnpm-workspace.yaml` 被自动追加了一条 `minimumReleaseAgeExclude: ['@changfenhuang/dsh-genui@0.14.0']`。
  看到 "minimum release age" 报错就是这个原因，等 24h 或保留这条排除项。

### 4. 侧边栏：`dsh-better-sidebar`

右侧栏 + 底部工作台：文件树（git 颜色、搜索、拖拽上传）、可编辑的 CodeMirror、
Markdown+Mermaid 预览、Git 面板（含只读提交图）、任务/子代理拓扑、终端、Side Chat。
另外向所有插件开放 `ctx.betterSidebar` 服务（`registerTab` / `registerFileViewer`）。

配置（`id: better-sidebar`，大部分偏好在界面里改并存在这个挂载行上）：

```yaml
- id: better-sidebar
  name: "dsh-better-sidebar"
  config:
    listLimit: 200        # 文件树单层最多读多少行
```

踩坑：

- **版本必须对宿主**。它的 npm `latest` 已经指到 0.28.0（要 DSH ≥0.2.1-rc.1），
  而 DSH 自己的 `latest` 还停在 0.2.0-rc.2 —— 在老宿主上直接 `@latest` 会装上不能用的版本。
  按上面的表钉版本，本机是 `0.28.0`。
- **"两个侧边栏"＝挂了两次**。如果 profile 的 `cordis.patch.yml` 里还留着老的
  `- insert: [{ id: better-sidebar, name: dsh-better-sidebar }]`，删掉它（bundle 已经负责 insert）。
  插件自己带了 `!!js` 守卫：别的 bundle（比如聚合包）已经挂了同名包时，这一行会自动 `disabled`。
- **`fs` 路由从 v0.23.0 起不做工作区包含校验**，宿主用户能读能写的路径它都能读写，
  只靠操作系统权限兜底；把它当成"等同本地 shell 权限"看待。
- 终端是 DSH 自己提供的（这个插件不带 PTY）。底部面板提示
  "host provides no terminal capability" 说明该部署没挂 terminal controller。
- pnpm 11 报 `Ignored build scripts` 时，到 `~/.dsh/profiles/desktop` 里跑 `pnpm approve-builds`。
- 升级后偏好"丢了"其实是搬到了挂载行上（老 `settings.yaml` 会迁移一次，不会反向覆盖）。

### 5. 通知提醒：`dsh-notify-me`

包名 `github:chromoany/dsh-notify-me`（v1.17.0，MIT）。让你离开 DSH 页面时也能知道 agent 何时
需要你裁决（审批 / 方案确认 / 提问）或后台回复跑完：系统通知 + 提示音 + 标签页标题标记。审批通知
还带「同意 / 拒绝」按钮（Web 端走带按钮的系统通知；桌面端点通知回到对话里裁决）。

配置走 **设置 → Notify me**（首次安装后刷新一次页面即可看到）。主要开关：启用提醒（总开关）、
系统通知 / 提示音 / 音量、页面打开时也提醒「需要你」（默认开）、页面打开时也提醒「回复完成」
（默认关）、当前对话不弹通知（默认开）、子代理会话不提醒（默认开）、点击通知回到对应对话
（默认开）。高级设置里还有音效来源、外部推送（Discord / Bark / Gotify / 飞书群机器人 / Server酱 /
WxPusher / 自定义 Webhook）、运行环境与诊断。

踩坑：
- **macOS 首次通知可能被系统静默**：系统设置里的「专注模式」「定时摘要」会把横幅收进摘要；
  若一条都看不到，先检查 系统设置 → 通知 → DeepSeek Harness。
- **桌面端点通知没有快捷裁决按钮**：点通知回到对话里裁决（Web 端才有按钮）。
- 多个 DSH 客户端同时在线会各弹一次（桌面窗口 + 浏览器标签页），跨客户端不重复。
- 若 profile 未装配 `dsh-client-ui-session`，插件退化为「只有设置卡、不发通知」，不报错。

### 通用踩坑

- **装完 host 侧立刻生效，客户端要硬刷新**。实测：三个插件装完，宿主侧的工具注册里马上就能看到
  `computer`、`render_ui`、`validate_dsh_ui`（`genui` 技能也立刻出现在技能目录里），
  但**已经打开的那个页面**里还查不到 `betterSidebar` 客户端服务——界面必须刷新一次才出现
  （桌面版在窗口里按 `Ctrl+Shift+R`）。这条对 better-sidebar / genui / computer-use 的设置页都成立。
- **命令行工具认 entry id，不认包名**。用 agent 的 plugin 管理工具做 `set_plugin` / 停用时，
  `target` 要写 `include:copylee-computer-use`、`include:genui`、`include:better-sidebar`、
  `include:dsh-web-tools`；写包名会报 `unknown-plugin`。
- **改了 `cordis.patch.yml` 要重启**；界面里的设置项是热写入（volatile），保存即生效，不用重启。
- **插件"装了但没反应"先看三件事**：宿主版本是否满足 `dsh.engines.dsh`、profile 是不是搞混了
  （`desktop` vs `web`）、页面有没有刷新。
- 代理环境下 `github:` 依赖可能拉不动。`dsh-web-tools` 只有 GitHub 源，先把 pnpm/HTTP_PROXY 配好
  （见 `dev_env.md` 的代理章节），再装。
