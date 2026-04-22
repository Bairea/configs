# Programming Environment Setup

## Rust

```bash
# 替换为中科大源
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup

# 安装 rust-analyzer
rustup component add rust-analyzer
```

## Go

```bash
# 替换为阿里云源
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOSUMDB=off

# 安装 gopls
go install golang.org/x/tools/gopls@latest
```

## Node.js

```bash
# 替换为淘宝源
npm config set registry https://registry.npmmirror.com
```

## Python (uv)

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 替换为清华源
export UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple

# 全局安装 Python 3.13
uv python install 3.13
uv python global 3.13

# 使用 uv tool install 安装工具
uv tool install isort
uv tool install black
uv tool install pyright
```

或者将源配置写入 `~/.config/uv/uv.toml`:

```toml
[python]
default-version = "3.13"

[install]
python-preference = "managed"

[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
```

## Claude Code 插件

```bash
# 安装 Claude Code CLI
npm install -g @anthropic-ai/claude-code

# 安装 MCP 插件 (需指定命令或 URL)
claude mcp add pyright-lsp
claude mcp add rust-analyzer-lsp
claude mcp add gopls-lsp

# 安装 ccstatusline (全局 npm)
npm install -g ccstatusline

# 安装 claude-mem (全局 npm)
npm install -g claude-mem
```

## 已知问题

- `ccstatusline`: `npx -y ccstatusline@latest` 执行超时，应使用 `npm install -g ccstatusline`
- `claude-mem`: `npx claude-mem install` 执行超时，应使用 `npm install -g claude-mem`
