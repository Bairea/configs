
更改 `~/.pi/agent/models.json` 和 `~/.pi/agent/settings.json` 。

```JSON
# ~/.pi/agent/models.json
{
  "providers": {
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
          "input": ["text"],
          "reasoning": false,
          "cost": {
            "input": 1,
            "output": 2,
            "cacheRead": 0.02,
            "cacheWrite": 0
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

