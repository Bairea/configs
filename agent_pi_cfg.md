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
    "longcat": {
      "baseUrl": "https://api.longcat.chat/anthropic",
      "api": "anthropic-messages",
      "apiKey": "$LONGCAT_API_KEY",
      "authHeader": true,
      "models": [
        {
          "id": "LongCat-2.0",
          "name": "LongCat-2.0",
          "contextWindow": 1000000,
          "input": [
            "text"
          ],
          "reasoning": true,
          "cost": {
            "input": 2,
            "output": 8,
            "cacheRead": 0.04,
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
    },
    "opencode-go": {
      "baseUrl": "https://opencode.ai/zen/go/v1",
      "api": "openai-completions",
      "apiKey": "$OPENCODE_GO_API_KEY",
      "models": [
        {
          "id": "deepseek-v4-flash",
          "name": "deepseek-v4-flash",
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

