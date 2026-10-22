# DeepSeek Harness (dsh) 模型配置

dsh 的模型供应商由 `@deepseek-ai/dsh-llm-pi-ai` 插件提供，配置写在 profile 的 patch 层
`~/.dsh/profiles/web/cordis.patch.yml`（顶层是 loader patch 数组，按插件 `id` 覆盖配置）。

- `apiKeyEnv` 只写凭据引用名，密钥本身存 `~/.dsh/.credentials.yaml`。
- 手工声明的路由必须给出 `api`、`baseURL` 与非空的 `models` 列表。
- `reasoningEfforts` 的键是选择器里显示的等级，值是该等级在协议中实际发送的拼写。
- `retryPolicy` 由 `dsh-llm-retry` 执行，省略时默认 normal 模式、最多重试 5 次。

```yaml
# ~/.dsh/profiles/web/cordis.patch.yml
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

改完 `cordis.patch.yml` 需要重启 dsh web，再在浏览器里硬刷新（Ctrl+Shift+R），模型选择器才会出现新路由。
