# Tmux Installation Guide

## TPM (Tmux Plugin Manager) Installation

### Step 1: Clone TPM Repository

Run the following command to install TPM:

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### Step 2: Configure .tmux.conf

Add the following lines to your `~/.tmux.conf` file:

```bash
# Initialize TPM (must be at the bottom of your .tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
```

### Step 3: Reload Tmux Configuration

Reload your tmux configuration by pressing `prefix + I` (default prefix is `Ctrl+b`).

Or from command line:

```bash
tmux source-file ~/.tmux.conf
```

## Installing Plugins

To install plugins, add the plugin to your `.tmux.conf`:

```bash
set -g @plugin 'tmux-plugins/plugin-name'
```

Then press `prefix + I` to install the plugin.

## Recommended Plugins

- `tmux-plugins/tpm` - Plugin manager
- `tmux-plugins/tmux-resurrect` - Save and restore tmux sessions
- `tmux-plugins/tmux-continuum` - Automatic session saving and restoring

## Usage

### Key Bindings

| Key | Description |
|-----|-------------|
| `prefix + I` | Install new plugins |
| `prefix + U` | Update plugins |
| `prefix + alt + u` | Uninstall plugins |

### Resurrect Commands

| Key | Description |
|-----|-------------|
| `prefix + Ctrl+s` | Save session |
| `prefix + Ctrl+r` | Restore session |

## Troubleshooting

If plugins are not loading, make sure:
1. TPM is installed at `~/.tmux/plugins/tpm`
2. The `run` command is at the bottom of `.tmux.conf`
3. Reload tmux configuration after changes

## Shell Aliases

Add the following to your `~/.bashrc`:

```bash
alias tn="tmux new -s"
# alias tk="tmux kill-session -t"
alias tl="tmux list-session"
alias ta="tmux a -t"
```

## Multi-Window Configuration

Add this function to `~/.bashrc` for a pre-configured multi-pane tmux session:

```bash
tmux3() {
    local session_name="${1:-work}"

    # 1. 检查 Session 是否已存在
    if tmux has-session -t "$session_name" 2>/dev/null; then
        tmux attach-session -t "$session_name"
        return 0
    fi

    # 2. 创建新 Session 并获取初始窗口的 ID
    # 使用 -P 和 -F 可以直接返回新创建的窗口 ID，无视 base-index 配置
    local window_id=$(tmux new-session -d -s "$session_name" -P -F '#{window_id}')

    # 3. 执行分屏 (使用窗口 ID 确保操作对象准确)
    # 第一次左右分：创建右侧窗格
    tmux split-window -h -t "$window_id"
    # 第二次上下分：在当前右侧窗格（即最后一个创建的窗格）下方切分
    tmux split-window -v -t "$window_id"

    # 4. 设置命令 (使用窗格索引的相对偏移或布局位置)
    # 这里的 .{top-left} 等是 Tmux 内置的定位符，不受 base-index 影响

    # 左侧窗格 (Top-Left)
    tmux send-keys -t "${window_id}.top-left" 'claude' C-m

    # 右上窗格 (Top-Right)
    tmux send-keys -t "${window_id}.top-right" 'lazygit' C-m

    # 右下窗格 (Bottom-Right)
    tmux send-keys -t "${window_id}.bottom-right" 'yazi' C-m

    # 5. 可选：调整布局
    # 例如：平分布局或设置比例
    # tmux select-layout -t "$window_id" main-vertical

    # 最后进入 session
    tmux attach-session -t "$session_name"
}
```

Usage: `tmux3 [session_name]` - creates a 3-pane tmux session with `claude`, `lazygit`, and `yazi`. Default session name is `work`.
