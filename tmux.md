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
