# dotfiles

个人开发环境配置，由 [chezmoi](https://chezmoi.io) 管理。

## 包含内容

| 文件 | 说明 |
|---|---|
| `.zshenv` | XDG 路径、Atuin 配置来源、所有 zsh 可用的工具 PATH；不导出 API Key |
| `.zprofile` | Homebrew 登录环境，并在系统 path_helper 之后恢复 PATH 优先级 |
| `.zshrc` | 交互补全、快捷键、历史过滤和工具初始化；补全缓存由此后台编译 |
| `.local/share/zsh/completions/` | 静态命令补全 |
| `.vimrc` | Vim 配置 |
| `.config/ghostty/config` | Ghostty 终端，50 MiB 回滚缓冲、关闭任务确认 |
| `.config/atuin/` | Atuin 历史搜索及主题 |
| `.config/starship.toml` | Starship 提示符 |
| `.config/fzf/fzfrc` | fzf 选项与配色 |
| `.config/bat/config` | bat 语法高亮 |
| `.config/ripgrep/rc` | ripgrep 搜索默认参数 |
| `.config/btop/` | btop 监控及 Snazzy 配色 |
| `.config/glow/glow.yml` | Markdown 预览 |
| `.config/git/config` + `ignore` | Git、delta 分页器、VS Code difftool 和全局忽略 |
| `.ssh/config` | SSH 配置 |
| `.config/Brewfile` | Homebrew 软件清单 |

`.zlogin`、yazi、Claude Code 配置和 Rime 词库目前不在管理范围。

## 恢复环境

先安装 Homebrew 和 chezmoi。若沿用现有 Git SSH 签名，还需安装、登录 1Password；使用 VS Code difftool 需确保 `code` 命令可用。

```sh
brew install chezmoi
chezmoi init git@github.com:tiancheng92/dotfiles.git
chezmoi diff
chezmoi apply
brew bundle --file="$HOME/.config/Brewfile"
```

打开新终端后，Zinit 会安装缺少的插件。Node.js 由 fnm 管理；非交互任务如需 Node.js，应在任务中显式选择版本，例如 `fnm exec --using=<version> node ...`。

API 凭据由各工具自己的凭据机制提供，不写入或全局导出到 `.zshenv`。恢复配置不需要通过 1Password CLI 渲染 shell 密钥。

## 日常维护

```sh
# 将已管理文件的本机修改同步到本地 chezmoi 源目录
chezmoi re-add ~/.zshenv ~/.zprofile ~/.zshrc ~/.config/starship.toml

# 确认源目录与当前配置一致
chezmoi status
chezmoi verify

# 检查本地源码改动；需要上传时再使用已有助手提交和推送
chezmoi-push -n -a
chezmoi-push ~/.zshrc ~/.config/starship.toml

# 预览更新步骤，不执行升级或重写 Brewfile
update-all --dry-run

# 更新 Homebrew、Zinit、独立 CLI，随后重新生成 Brewfile
update-all
```

`.zshenv` 现在对应普通源文件 `dot_zshenv`，可直接 `chezmoi re-add`。如果以后新增包含秘密的模板，仍应直接编辑模板，不把渲染后的凭据加入仓库。

`update-all` 优先通过 `~/.local/bin/codegraph upgrade` 更新独立 Codegraph，并对已安装工具执行 `go install golang.org/x/tools/gopls@latest` 和 `rustup update --no-self-update`。交互 shell 中的 `~/.local/bin` 同样优先于 fnm 的全局包 shim，Node 版本仍由 fnm 管理。gopls 更新到当前可执行文件所在目录，Homebrew 管理的 gopls 则跳过；Rust 只更新已安装工具链，rustup 本身仍由 Homebrew 管理。未安装的工具会跳过，不自动新增 CLI。RTK、uv、Atuin 等继续由 Homebrew 更新。某一步失败会继续其余步骤，最后汇总失败项并返回非零状态。

`zinit-update` 从磁盘安装目录枚举插件，包括尚未延迟加载的插件；仅 self-update 一次，然后逐个调用 Zinit 更新以保留原生 hooks。临时 Git 包装器额外捕获 fetch/pull 等失败，弥补上游丢失退出状态的问题；不自动删除任何插件。未来若自定义 hook 自行忽略错误，仍需由该 hook 保证正确返回状态。

`chezmoi-push` 将提交与推送分开，干净工作树也能重试上次失败的推送；不自动设置 remote/upstream。`-a` 包括文件及符号链接，链接保留自身路径并用 chezmoi add 更新，模板跳过。所有目标先预检，`-n` 不同步、不提交、不推送。

Atuin 固定使用 `.config/atuin`；不要再创建另一份 `~/.atuin/config.toml` 覆盖它。账户和历史数据保持既有位置。Atuin 与原生 zsh 历史使用对应的敏感命令过滤规则，逐行检查多行命令，并保留前导空格/Tab 不记录的约定；调整策略时要同时更新 `history_filter` 和 `_zsh_filter_sensitive_history`。

Ctrl+Y 使用 NUL 分隔完整历史记录，支持多行复制。输入行中的 Ctrl+Z 暂存当前编辑内容，执行一条临时命令后恢复；前台程序的 Ctrl+Z 作业控制仍由终端处理。`find`、`ps`、`df` 保留原生命令，替代工具直接使用 `fd`、`procs`、`duf`。Ctrl+T 不再全局排除 dist/build，仍尊重项目 ignore。

全局 Git ignore 保留系统临时文件、明确的依赖缓存和已有 `.env` 保护；团队配置、`vendor`、`dist`、`build` 等由项目 `.gitignore` 决定。项目也应自行声明依赖忽略规则，避免依赖个人全局配置。

ripgrep 默认仍尊重 Git ignore。排查被忽略的源码或产物时使用 `rg --no-config --no-ignore`，如需隐藏文件再加 `--hidden`，并限定检索目录。
