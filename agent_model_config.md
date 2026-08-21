

### Crush

```JSON
# ~/.config/crush/crush.json
{
  "$schema": "https://charm.land/crush.json",
  "providers": {
    "xfyun": {
      "type": "openai-compat",
      "base_url": "https://maas-coding-api.cn-huabei-1.xf-yun.com/v2",
      "api_key": "$XF_API_KEY",
      "models": [
        {
          "id": "xopglm5",
          "name": "GLM-5",
          "context_window": 200000,
          "default_max_tokens": 131072,
          "cost_per_1m_in": 1,
          "cost_per_1m_out": 2,
          "cost_per_1m_in_cached": 0.02,
          "cost_per_1m_out_cached": 0,
          "can_reason": true
        },
        {
          "id": "xopglm51",
          "name": "GLM-5.1",
          "context_window": 200000,
          "default_max_tokens": 131072,
          "cost_per_1m_in": 1,
          "cost_per_1m_out": 2,
          "cost_per_1m_in_cached": 0.02,
          "cost_per_1m_out_cached": 0,
          "can_reason": true
        }
      ]
    },
    "deepseek": {
      "type": "openai-compat",
      "base_url": "https://api.deepseek.com",
      "api_key": "$DEEPSEEK_API_KEY",
      "models": [
        {
          "id": "deepseek-v4-pro",
          "name": "DeepSeek-V4-Pro",
          "context_window": 1048576,
          "default_max_tokens": 384000,
          "cost_per_1m_in": 1,
          "cost_per_1m_out": 2,
          "cost_per_1m_in_cached": 0.02,
          "cost_per_1m_out_cached": 0,
          "can_reason": true
        },
        {
          "id": "deepseek-v4-flash",
          "name": "DeepSeek-V4-Flash",
          "context_window": 1048576,
          "default_max_tokens": 384000,
          "cost_per_1m_in": 3,
          "cost_per_1m_out": 6,
          "cost_per_1m_in_cached": 0.025,
          "cost_per_1m_out_cached": 0,
          "can_reason": true
        }
      ]
    }
  },
  "models": {
    "large": {
      "provider": "xfyun",
      "model": "xopglm5",
      "reasoning_effort": "high"
    },
    "small": {
      "provider": "xfyun",
      "model": "xopglm5",
      "reasoning_effort": "medium"
    }
  }
}

```

### Pi agent
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

# ~/.pi/agent/settings.json
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

