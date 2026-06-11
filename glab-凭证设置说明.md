# glab 安装与 GitLab 免密凭证配置

## 1. 安装 Go（仅首次）

```bash
# 安装 Go（Debian 12）
sudo apt-get install -y golang-go

# 如版本过低，手动下载新版
curl -sL https://go.dev/dl/go1.26.4.linux-amd64.tar.gz -o /tmp/go.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf /tmp/go.tar.gz
export PATH="/usr/local/go/bin:$PATH"
```

## 2. 编译安装 glab

```bash
git clone https://gitlab.com/gitlab-org/cli.git /tmp/cli
cd /tmp/cli
export PATH="/usr/local/go/bin:$PATH"
make build
make install

# 永久加入 PATH
echo 'export PATH="$PATH:/tmp/cli/bin"' >> ~/.bashrc
source ~/.bashrc
glab version
```

## 3. 配置 glab 认证

```bash
# 将 GitLab Personal Access Token 写入 glab 配置
glab auth login --hostname gitlab.cent.hz --token <你的-token>
```

编辑 `~/.config/glab-cli/config.yml`，确保 `gitlab.cent.hz` 段包含：

```yaml
    gitlab.cent.hz:
        api_protocol: http          # 内网 GitLab 走 HTTP
        token: glpat-xxxxxxxxxxxx   # 你的 Personal Access Token
```

验证：

```bash
glab auth status --hostname gitlab.cent.hz
# ✓ Logged in to gitlab.cent.hz as <用户名>
```

## 4. 配置 Git 免密推送

```bash
# 将 token 写入 git 凭证文件
echo "protocol=http
host=gitlab.cent.hz
username=oauth2
password=<你的-token>" | git credential-store --file ~/.git-credentials-glab store
```

```bash
# 全局启用该凭证文件
git config --global credential.helper 'store --file ~/.git-credentials-glab'
```

验证：

```bash
# 测试克隆（应免密）
cd /tmp && git clone http://gitlab.cent.hz/st/clustering.git test-clone
```

## 5. 日常使用

所有项目均免密操作：

```bash
git add .
git commit -m "feat: your change"
git push            # 自动读取 token，无需输入密码
```

## 说明

| 配置 | 路径 | 说明 |
|------|------|------|
| glab 配置文件 | `~/.config/glab-cli/config.yml` | 存储 token，供 `glab` CLI 使用 |
| Git 凭证文件 | `~/.git-credentials-glab` | 存储 token，供 `git push/clone` 使用 |
| Git 全局配置 | `credential.helper` | 指向凭证文件，所有仓库生效 |

注意：token 有权限范围，如需要推送其他项目，确保 token 勾选了 `api` 和 `write_repository` 权限。
