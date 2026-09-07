# Vim 配置

这是面向 Linux 桌面/终端 Vim 的个人配置，主配置文件为仓库根目录的
`.vimrc`。配置基于较早版本的 amix Vim 配置，保留了 Taglist、ack.vim
和 ack 的使用方式。

## 安装

```bash
cp .vimrc ~/.vimrc
```

安装 Vim、Taglist、ack.vim 和 ack。以 Debian/Ubuntu 为例：

```bash
sudo apt install vim ack
```

然后将 `ack.vim` 放到 Vim 插件目录，例如：

```bash
mkdir -p ~/.vim/plugin
cp ack.vim ~/.vim/plugin/
```

Taglist 也应安装到 Vim 的插件目录。若使用 Vim 8 或更高版本，推荐将
插件放到 `~/.vim/pack/vendor/start/` 下，或使用现有的 Vim 插件管理器。

依赖检查：

```vim
:echo exists(':Ack')
:echo exists(':TlistOpen')
:echo executable('ack')
```

三个结果都应为 `1`。其中 `:Ack` 由 ack.vim 提供，`:TlistOpen` 由
Taglist 提供，`ack` 则必须能在 Linux 的 `PATH` 中执行。

## 常用功能

配置使用逗号作为 `<Leader>`：

| 按键 | 功能 |
| --- | --- |
| `,w` | 强制保存当前文件 |
| `,bd` | 关闭当前 buffer，不关闭窗口 |
| `,ba` | 强制关闭所有 buffer |
| `,tn` / `,tc` | 新建/关闭 tab |
| `,te` | 在当前文件目录打开新 tab |
| `,cd` | 将工作目录切换到当前文件目录 |
| `,g` | 使用 ack.vim 开始搜索 |
| `,cc` | 打开 quickfix 窗口 |
| `,n` / `,p` | 下一个/上一个 quickfix 结果 |
| `,ss` | 切换拼写检查 |
| `,pp` | 切换 paste 模式 |
| `,` + Enter | 清除搜索高亮 |

在可视模式下，`*` 和 `#` 会搜索选中文本，`gv` 会使用 Ack 搜索选中文本，
`,r` 会生成针对选中文本的替换命令。执行替换前应先检查命令内容。

## Linux 范围内的检查结论

本次按“Linux + Taglist + ack.vim + ack”作为目标环境重新检查，未直接修改
`.vimrc`。原配置在该范围内不存在必须立即修复的 Windows 兼容问题；其中
`:W` 使用 `sudo tee`，在 Linux 上是合适的：

```vim
:W
```

建议后续按以下优先级优化。

### 建议优先处理

1. **恢复能力**

   当前同时关闭了 backup、写备份和 swap：

   ```vim
   set nobackup
   set nowb
   set noswapfile
   ```

   这会失去 Vim 崩溃后的恢复能力。建议至少启用 swap，并增加持久化 undo：

   ```vim
   set swapfile
   set undofile
   set undodir^=$HOME/.vim/undo//
   ```

   使用前应创建目录：

   ```bash
   mkdir -p ~/.vim/undo
   ```

2. **限制强制删除 buffer**

   `,ba` 使用 `bd!`，会丢弃所有未保存修改。建议改成不带 `!` 的版本，
   让 Vim 在存在未保存内容时阻止操作，确实需要时再手动执行强制删除。

3. **避免自动命令重复注册**

   `BufReadPost`、`BufWrite` 自动命令没有放进 `augroup`。如果配置被
   `:source ~/.vimrc` 多次执行，命令可能重复注册。建议使用专用
   `augroup`，并在组内先执行 `autocmd!`。

### 建议按需要处理

4. **补充 ack 搜索参数**

   当前 `,g` 只映射到 `:Ack`，搜索范围和忽略规则交给 ack.vim/ack 默认值。
   可以在 `g:ackprg` 中统一设置参数，例如排除构建目录、依赖目录和版本控制
   目录；具体参数应结合项目类型决定，避免把 `ack` 的默认行为改得过于隐蔽。

5. **Taglist 自动打开**

   `Tlist_Auto_Open=1` 会在每次启动 Vim 时自动打开 Taglist。若经常在终端中
   编辑小文件，这会占用窗口空间；可以改为手动打开 Taglist，只在浏览大型
   源码时执行 `:TlistOpen`。

6. **降低全局映射副作用**

   `map j gj`、`map k gk`、`map <Space> /`、`map 0 ^` 等映射会覆盖 Vim
   默认行为，而且 `map` 是递归映射。建议逐步改为限定模式的 `nnoremap`，
   例如只在普通模式映射 `j`/`k`，减少对插件和插入模式的影响。

7. **按文件类型设置缩进**

   全局固定四空格适合作为个人默认值，但不适合所有项目。建议保留全局默认，
   再通过 `after/ftplugin/` 为 Python、Make、Go 等语言设置各自的
   `tabstop`、`shiftwidth` 和 `expandtab`。

8. **行尾空格清理范围**

   当前只在保存 Python 和 CoffeeScript 文件时删除行尾空格。建议用
   `BufWritePre` + `augroup` 管理，并确认项目是否允许行尾空格；如果项目有
   格式化工具，应避免 Vim 与格式化工具重复修改同一文件。

9. **现代 Vim 写法**

   `set viminfo^=%`、`set t_Co=256` 和 `set t_vb=` 来自较旧 Vim 配置。
   Linux 终端下通常仍可工作，但在 Vim 8/9 或 Neovim 中建议分别评估：
   使用 `:set viminfo?`/`shada` 检查版本差异，并使用终端自身的颜色能力，
   不再强制设置过时的终端选项。

10. **减少不必要的旧配置**

    `set encoding=utf8` 在现代 Vim 中通常已经是默认值，`set magic` 也通常
    不需要显式设置。它们可以保留以表达意图，但后续整理时可删除冗余项。

## 当前状态

- 已将附件原样保存为根目录 `.vimrc`。
- 按 Linux 使用范围完成静态检查。
- 本次没有应用上述优化建议，避免改变现有按键和编辑行为。
- 当前环境未提供可执行的 Vim/Neovim，因此未进行实际 `vim -Nu .vimrc`
  启动检查；在 Linux 主机上建议用 `vim -Nu ~/.vimrc` 启动后检查
  `:messages`、`:scriptnames` 和上述依赖命令。