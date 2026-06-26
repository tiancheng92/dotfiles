# dotfiles

个人开发环境配置，由 [chezmoi](https://chezmoi.io) 管理。

## 包含内容

| 文件 | 说明 |
|---|---|
| `.zshrc` / `.zshenv` / `.zprofile` | zsh 配置（zshenv 含 API Key 模板） |
| `.vimrc` | vim 配置 |
| `.config/ghostty/config` | Ghostty 终端 |
| `.config/atuin/config.toml` | shell 历史（Atuin） |
| `.config/starship.toml` | 提示符（Starship） |
| `.config/bat/config` | bat 语法高亮 |
| `.config/ripgrep/rc` | ripgrep 搜索 |
| `.config/git/config` + `ignore` | git 全局配置 |
| `.ssh/config` | SSH 配置 |
| `.config/Brewfile` | Homebrew 软件清单 |
| `.claude/settings.json` | Claude Code 配置 |
| `Library/Rime/*.custom.yaml` | 鼠须管输入法定制 |
| `Library/Rime/custom_phrase.txt` | 自定义词组 |

## 依赖准备

恢复前确保已安装：

- [Homebrew](https://brew.sh)
- [1Password](https://1password.com) + [1Password CLI](https://developer.1password.com/docs/cli/)（用于读取 API Key）
- [chezmoi](https://chezmoi.io)（`brew install chezmoi`）

## 恢复步骤

### 1. 初始化 chezmoi 并应用配置

```bash
chezmoi init --apply git@github.com:tiancheng92/dotfiles.git
```

chezmoi 会自动从 1Password 读取 API Key 并写入 `.zshenv`，需要在 1Password 中登录授权一次。

### 2. 配置 chezmoi 1Password 集成

```bash
mkdir -p ~/.config/chezmoi
cat > ~/.config/chezmoi/chezmoi.toml << 'EOF'
[onepassword]
  command = "op"
EOF
```

### 3. 安装 Homebrew 软件

```bash
brew bundle --file=~/.config/Brewfile
```

### 4. 安装 zsh 插件

打开终端，zinit 会自动安装所有插件（首次启动会稍慢）。

### 5. 配置 Claude Code MCP

```bash
# Jina MCP（全局）
claude mcp add --transport http -s user jina https://mcp.jina.ai/v1 \
  --header "Authorization: Bearer ${JINA_API_KEY}"
```

### 6. 恢复鼠须管词库

在新机器上：
1. clone [rime-ice](https://github.com/iDvel/rime-ice) 到 `~/Library/Rime/`
2. 在 `~/Library/Rime/installation.yaml` 中添加：
   ```yaml
   sync_dir: "/Users/用户名/Library/Mobile Documents/com~apple~CloudDocs/RimeSync"
   ```
3. 鼠须管菜单 → **同步用户数据**，从 iCloud 恢复词库

## 日常维护

```bash
# 配置有变更后同步到 GitHub
chezmoi re-add <修改的文件>
cd ~/.local/share/chezmoi
git add -A && git commit -m "chore: update" && git push

# 更新软件清单
brew bundle dump --file=~/.config/Brewfile --force
chezmoi re-add ~/.config/Brewfile
cd ~/.local/share/chezmoi && git add -A && git commit -m "chore: update Brewfile" && git push
```
