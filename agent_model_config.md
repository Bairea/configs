

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

```JSON
# ~/.pi/agent/models.json
{
  "providers": {
    "opencode-go": {
      "baseUrl": "https://opencode.ai/zen/go/v1",
      "api": "openai-completions",
      "apiKey": "$OPENCODE_GO_API_KEY"
    },
    "xfyun": {
      "baseUrl": "https://maas-coding-api.cn-huabei-1.xf-yun.com/v2",
      "api": "openai-completions",
      "apiKey": "!echo $XF_API_KEY",
      "models": [
        {
          "id": "xopglm5",
          "name": "GLM-5",
          "contextWindow": 200000,
          "maxTokens": 131072,
          "input": [
            "text"
          ],
          "reasoning": false,
          "cost": {
            "input": 1,
            "output": 2,
            "cacheRead": 0.02,
            "cacheWrite": 0
          }
        },
        {
          "id": "xopglm51",
          "name": "GLM-5.1",
          "contextWindow": 200000,
          "maxTokens": 131072,
          "input": [
            "text"
          ],
          "reasoning": false,
          "cost": {
            "input": 1,
            "output": 2,
            "cacheRead": 0.02,
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
        }
      ]
    }
  }
}

# ~/.pi/agent/settings.json
{
  "lastChangelogVersion": "0.74.1",
  "defaultProvider": "xfyun",
  "defaultModel": "xopglm5",
  "hideThinkingBlock": true,
  "quietStartup": true,
  "theme": "dark"
}
```

