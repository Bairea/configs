### Pi agent
1. 安装
```Shell
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

2. 插件推荐：
```Shell
pi install npm:pi-subagents
pi install npm:pi-mcp-adapter
pi install npm:pi-open-tui
pi install npm:pi-rtk-optimizer
pi install npm:pi-web-access
```
`pi-web-access`插件，可以在`~/.pi/web-search.json`中配置对应的key，可见[nicobailon/pi-web-access: Web search and content extraction extension for Pi coding agent](https://github.com/nicobailon/pi-web-access#install)

3. llm api
更改 `~/.pi/agent/models.json` 和 `~/.pi/agent/settings.json` 。
有一些供应商直接设置对应的环境变量就行，比如 OPENCODE_API_KEY，https://pi-doc.com/docs/latest/providers#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E6%88%96-auth-%E6%96%87%E4%BB%B6；
暂时未发现能支持 openai-response，对于一些供应商，此时使用anthropic-messages的api设置。

```JSON
# ~/.pi/agent/models.json
{
  "providers": {
    "zy-api": {
      "baseUrl": "https://image.yydsapi.uno",
      "api": "anthropic-messages",
      "apiKey": "$ZYAPI_API_KEY",
      "authHeader": true,
      "compat": {
        "supportsEagerToolInputStreaming": false,
        "allowEmptySignature": true,
        "supportsLongCacheRetention": false
      },
      "models": [
        {
          "id": "grok-4.6",
          "name": "Grok 4.6",
          "reasoning": true,
          "input": ["text", "image"],
          "contextWindow": 500000,
          "maxTokens": 16384
        },
        {
          "id": "grok-4.7",
          "name": "Grok 4.7",
          "reasoning": true,
          "input": ["text", "image"],
          "contextWindow": 500000,
          "maxTokens": 128000,
          "thinkingLevelMap": {
            "minimal": "minimal",
            "low": "low",
            "medium": "medium",
            "high": "high",
            "xhigh": "xhigh",
            "max": null
          },
          "cost": {
            "input": 2,
            "output": 6,
            "cacheRead": 0.5,
            "cacheWrite": 0
          }
        }
      ]
    },
    "open-design": {
      "baseUrl": "https://amr-link.open-design.ai/v1",
      "api": "openai-completions",
      "apiKey": "$OD_API_KEY",
      "models": [
        {
          "id": "deepseek-v4.1-flash",
          "name": "DeepSeek V4.1 Flash",
          "contextWindow": 1000000,
          "maxTokens": 384000,
          "input": ["text", "image"],
          "reasoning": true,
          "compat": {
            "requiresReasoningContentOnAssistantMessages": true,
            "thinkingFormat": "deepseek",
            "supportsDeveloperRole": false,
            "maxTokensField": "max_tokens",
            "supportsReasoningEffort": true,
            "reasoningEffortMap": {
              "minimal": "high",
              "low": "high",
              "medium": "high",
              "high": "max",
              "xhigh": "max"
            }
          },
          "cost": {
            "input": 0.3,
            "output": 1.20,
            "cacheRead": 0.006,
            "cacheWrite": 0
          }
        }
      ]
    },
    "deepseek": {
      "baseUrl": "https://api.deepseek.com",
      "api": "openai-completions",
      "apiKey": "$DEEPSEEK_API_KEY",
      "models": [
        {
          "id": "deepseek-v4-pro",
          "name": "DeepSeek V4 Pro",
          "contextWindow": 1000000,
          "maxTokens": 384000,
          "input": [
            "text"
          ],
          "reasoning": true,
          "cost": {
            "input": 3,
            "output": 6,
            "cacheRead": 0.025,
            "cacheWrite": 0
          },
          "compat": {
            "requiresReasoningContentOnAssistantMessages": true,
            "thinkingFormat": "deepseek",
            "reasoningEffortMap": {
              "minimal": "high",
              "low": "high",
              "medium": "high",
              "high": "max",
              "xhigh": "max"
            }
          }
        },
        {
          "id": "deepseek-v4-flash",
          "name": "DeepSeek V4 Flash",
          "contextWindow": 1000000,
          "maxTokens": 384000,
          "input": [
            "text"
          ],
          "reasoning": true,
          "cost": {
            "input": 1,
            "output": 2,
            "cacheRead": 0.02,
            "cacheWrite": 0
          },
          "compat": {
            "requiresReasoningContentOnAssistantMessages": true,
            "thinkingFormat": "deepseek",
            "reasoningEffortMap": {
              "minimal": "high",
              "low": "high",
              "medium": "high",
              "high": "max",
              "xhigh": "max"
            }
          }
        }
      ]
    }
  }
}
```

> **本机当前状态（2026-10-31 更新）**
> - `models.json` 实际只启用 `zy-api` / `open-design` / `deepseek`（`command-code`、`longcat`、`opencode-go` 已移出本机配置，需要时可从 git 历史加回）。
> - `zy-api.baseUrl` 已改为 `https://image.yydsapi.uno`（与 dsh 的 zy-api 同一 host）。
> - `open-design` 用 `OD_API_KEY` 环境变量，本机尚未导出，使用前需 `export OD_API_KEY=...`。
> - ⚠️ `settings.json` 的 `defaultProvider` 仍是 `opencode-go`，但 `opencode-go` 已不在 `models.json` 里，默认值悬空，pi 启动会找不到默认供应商。请改为 `deepseek` 或 `zy-api`，或重新加回 `opencode-go`。

默认设置，~/.pi/agent/settings.json
```
{
  "lastChangelogVersion": "0.74.1",
  "defaultProvider": "opencode-go",
  "defaultModel": "deepseek-v4-flash",
  "hideThinkingBlock": true,
  "defaultThinkingLevel": "xhigh",
  "quietStartup": true,
  "theme": "dark"
}
```

