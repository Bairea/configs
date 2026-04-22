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

## Yazi (Terminal File Manager)

Yazi 是一个用 Rust 编写的终端文件管理器。

```bash
# 方法1: 使用预编译二进制 (推荐)
curl -fsSL https://apt.fury.io/yazi-rs/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/yazi.gpg
echo "deb [signed-by=/usr/share/keyrings/yazi.gpg] https://apt.fury.io/yazi-rs/ /" | sudo tee /etc/apt/sources.list.d/yazi.list
sudo apt update && sudo apt install yazi

# 方法2: 使用 Cargo 安装
cargo install --locked yazi-fm yazi-cli

# 方法3: 从 GitHub Releases 下载
# https://github.com/sxyazi/yazi/releases
```

可选依赖 (用于增强功能):

```bash
sudo apt install ffmpeg 7zip jq poppler-utils fd-find ripgrep fzf zoxide imagemagick
```

## Lazygit (Terminal Git UI)

Lazygit 是一个简单的终端 Git 用户界面。

```bash
# 下载最新版本
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": *"v\K[^"]*')
LAZYGIT_ARCH=$(uname -m | sed -e 's/aarch64/arm64/')
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_${LAZYGIT_ARCH}.tar.gz"
tar xf lazygit.tar.gz lazygit
sudo install lazygit -D -t /usr/local/bin/
```
候选
```bash
go install github.com/jesseduffield/lazygit@latest
```

## GitHub CLI (gh)

GitHub CLI 是 GitHub 的官方命令行工具，用于管理仓库、问题、拉取请求等。

```bash
# 方法1: 使用包管理器安装 (推荐)
# Ubuntu/Debian
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# Fedora
sudo dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
sudo dnf install gh

# Arch Linux
sudo pacman -S github-cli

# 方法2: 使用脚本安装
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# 方法3: 手动下载二进制
# 从 GitHub Releases 下载: https://github.com/cli/cli/releases
```

安装后验证:
```bash
gh --version
gh auth login
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
