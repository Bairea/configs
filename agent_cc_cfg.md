
## Claude Code 插件

```bash
# 安装 Claude Code CLI
npm install -g @anthropic-ai/claude-code

# 安装 LSP 插件 (从官方 marketplace)
claude plugins install pyright-lsp
claude plugins install rust-analyzer-lsp
claude plugins install gopls-lsp
claude plugin install security-guidance@claude-plugins-official

# 安装 ccstatusline (全局 npm)
npm install -g ccstatusline

# 安装 claude-mem (全局 npm)
npm install -g claude-mem
```

```bash
# 手动升级
# 1. 查看当前全局 npm 安装路径（确认一下）
npm root -g

# 2. 删除旧的 claude-code 目录
rm -rf /home/dev/nodejs/lib/node_modules/@anthropic-ai/claude-code

# 3. 删除 npm 可能残留的临时目录
rm -rf /home/dev/nodejs/lib/node_modules/@anthropic-ai/.claude-code-*

# 4. 清理 npm cache（推荐）
npm cache clean --force

# 5. 重新安装
npm install -g @anthropic-ai/claude-code

# 6. 验证是否安装成功
claude --version
```

## RTK

RTK 是一个由 RTK-AI 开发的工具。

```bash
# 安装
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

# 添加到 PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc

# 初始化
rtk init -g
```

更多详情请参考:
- 快速安装: https://github.com/rtk-ai/rtk#quick-install-linuxmacos
- 快速开始: https://github.com/rtk-ai/rtk#quick-start

## 已知问题

- `ccstatusline`: `npx -y ccstatusline@latest` 执行超时，应使用 `npm install -g ccstatusline`
- `claude-mem`: `npx claude-mem install` 执行超时，应使用 `npm install -g claude-mem`

## 工具调用日志

Claude Code 可以通过 hook 机制记录工具调用日志。这样可以审计和追踪所有工具使用情况。

### 配置步骤

1. 在 `~/.claude/settings.json` 中添加 PostToolUse hook：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "async": true,
            "command": "/home/dev/.claude/hooks/audit-log.sh"
          }
        ]
      }
    ]
  }
}
```

2. 创建 hook 脚本 `~/.claude/hooks/audit-log.sh`：

```bash
#!/bin/bash
AUDIT_LOG="${CLAUDE_PROJECT_DIR:-${PWD}}/.claude/audit.log"

mkdir -p "$(dirname "$AUDIT_LOG")"
touch "$AUDIT_LOG"

read -r JSON_INPUT

TOOL_NAME=$(echo "$JSON_INPUT" | jq -r '.tool_name // "Unknown"')

# Extract useful values - priority: command > file_path > pattern > url > query
TOOL_INPUT=$(echo "$JSON_INPUT" | jq -r '
.tool_input | 
if .command then 
  "cmd=" + (.command | .[0:120]) + (if length > 80 then "..." else "" end)
elif .file_path then 
  "path=" + (.file_path | .[0:100])
elif .pattern then 
  "pattern=" + (.pattern | .[0:40])
elif .url then 
  "url=" + (.url | .[0:100])
elif .query then 
  "q=" + (.query | .[0:100])
elif type == "object" then
  . as $obj | ($obj | keys | .[0:5]) | map("\(.)=" + (.[0:40] // "")) | join(" ")
else 
  "—"
end
' 2>/dev/null)

[ -z "$TOOL_INPUT" ] && TOOL_INPUT="—"

TIMESTAMP=$(date +%Y-%m-%dT%H:%M:%S)
echo "$TIMESTAMP | $TOOL_NAME | $TOOL_INPUT" >> "$AUDIT_LOG"
```

3. 确保脚本有执行权限：

```bash
chmod +x ~/.claude/hooks/audit-log.sh
```

4. 日志将记录在项目目录的 `.claude/audit.log` 文件中，格式为：
```
TIMESTAMP | TOOL_NAME | TOOL_INPUT
```
