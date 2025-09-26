Zathura 是一款深受 Vim 用户喜爱的、键盘驱动的极简 PDF 查看器。它轻量、高效，支持多种文档格式。

### ⌨️ 一、常用快捷键操作

Zathura 的快捷键设计遵循 Vim 的风格，让你可以完全脱离鼠标。

| 操作类型         | 快捷键                 | 说明                                                                 |
| :--------------- | :--------------------- | :------------------------------------------------------------------- |
| **移动与导航**   | `h`, `j`, `k`, `l`     | 向左、下、上、右移动（或使用箭头键）                          |
|                  | `空格键`                 | 向下滚动半页（或 `Ctrl+f`）                                  |
|                  | `Shift+空格`           | 向上滚动半页（或 `Ctrl+b`）                                  |
|                  | `gg`                   | 跳转到文档第一页                                            |
|                  | `G`                    | 跳转到文档最后一页                                           |
|                  | `数字 + G`             | 跳转到指定页码（如 `10G` 跳到第 10 页）                          |
|                  | `<tab>键`              | 显示书签目录，点击后进行跳转                                   |
| **搜索**         | `/` + `关键词`         | 向后搜索关键词，按 `n` 下一个，`N` 上一个（类似 Vim 和 `less`） |
|                  | `?` + `关键词`         | 向前搜索关键词                                                |
| **缩放与调整**   | `+`                    | 放大页面                                                     |
|                  | `-`                    | 缩小页面                                                     |
|                  | `a`                    | 自动调整页面大小到适应窗口宽度或高度（取决于 `adjust-open` 设置） |
|                  | `s`                    | 仅调整页面到窗口宽度                                          |
| **链接操作**     | `f`                    | **高亮显示所有链接**并标注数字，输入数字后回车可用浏览器打开该链接 |
| **旋转**         | `r`                    | 顺时针旋转页面 90 度                                          |
|                  | `R`                    | 重新载入文档                                                  |
| **界面控制**     | `Ctrl+m`               | 显示/隐藏底部输入栏（Inputbar）                                |
|                  | `Ctrl+n`               | 显示/隐藏底部状态栏（Statusbar）                                |
|                  | `F5`                   | 切换全屏模式                                                  |
| **退出**         | `q` 或 `:q`            | 退出 Zathura                                         |
|                  | `Ctrl+q`               | 退出（在某些配置下）                                           |

### ⚙️ 二、常用配置（`~/.config/zathura/zathurarc`）

Zathura 的高度可定制性体现在其配置文件中。如果配置文件不存在，可以手动创建。

```bash
# 创建配置目录和文件
mkdir -p ~/.config/zathura
vim ~/.config/zathura/zathurarc  # 或用你喜欢的编辑器
```

以下是一些实用的配置示例（你可以根据需要取消注释并修改）：

```ini
# 设置默认字体和大小
set font "Noto Sans CJK SC Regular 10"

# 设置打开文件时的自适应方式 ("width" 或 "best")
set adjust-open "width"

# 复制到系统剪贴板（解决默认复制到主选择缓冲区的问题）
set selection-clipboard clipboard

# 界面选项：可设置为空以隐藏所有GUI元素，或按需添加
# 可选项目包括: statusbar, inputbar, horizontal-scrollbar, vertical-scrollbar
set guioptions ""

# 启用实时搜索（输入搜索词时即时高亮结果）
set incremental-search true

# 在窗口标题中只显示文件名（而非完整路径）
set window-title-basename true

# 显示垂直滚动条
set show-v-scrollbar true

# 设置默认窗口大小
set window-width 1300
set window-height 760
```
*配置修改后，重启 Zathura 生效。*

### 📑 三、高级功能与技巧

*   **命令模式**：按 `:` 进入命令模式，执行更复杂的操作。
    *   `:bmark <名称>` - 添加书签
    *   `:blist` - 列出所有书签
    *   `:bdelete <名称>` - 删除书签
    *   `:info` - 显示文档信息
    *   `:open another_file.pdf` - 来打开另一个文件

*   **标记功能**：类似 Vim，可以使用 `m` + `一个字母` 来标记当前位置，之后可以通过 `'` + `该字母` 快速跳转回来。

*   **自动重载**：当正在查看的文档被外部程序修改后（例如重新编译的 LaTeX PDF），Zathura 会自动刷新显示。

*   **书签功能**：按 `:` 进入命令模式，可以执行 `:bmark <名称>` 来添加书签，`:blist` 来列出所有书签。




### 🔧 四、安装PDF插件

在 Arch Linux 上，你需要安装一个特定的插件来让 Zathura 支持 PDF。最常用的是 `zathura-pdf-poppler`（基于 Poppler 库）或 `zathura-pdf-mupdf`（基于 MuPDF 库）。**推荐安装 `zathura-pdf-poppler`**：

```bash
sudo pacman -S zathura-pdf-poppler
```

安装完成后，**请完全关闭所有已打开的 Zathura 窗口**，然后重新尝试打开你的 PDF 文件。


---

### 📁 五、打开 PDF 文件

在终端中，使用以下命令格式即可用 Zathura 打开指定的 PDF 文件：
```bash
zathura /path/to/your/file.pdf
```
例如，打开你当前目录下的 `document.pdf` 文件：
```bash
zathura document.pdf
```

Zathura 启动后，你会看到一个**极简的界面**，没有复杂的菜单栏和按钮，让你能专注于阅读内容本身。

### ⚙️ 六、个性化配置（可选）

Zathura 的高度可定制性是其一大亮点。你可以通过修改配置文件 `~/.config/zathura/zathurarc` 来调整外观和行为。如果该文件或目录不存在，可以手动创建。

以下是几个实用的配置示例，你可以添加到配置文件中：

```bash
# 设置默认字体和大小
set font "Noto Sans CJK SC Regular 10"

# 设置打开文件时的自适应方式为按宽度适应
set adjust-open "width"

# 隐藏所有GUI元素（状态栏、输入栏等），获得更纯净的阅读体验
set guioptions ""

# 启用实时搜索（输入搜索词时即时高亮结果）
set incremental-search true

# 复制到系统剪贴板（而不仅仅是X11的主选择缓冲区）
set selection-clipboard clipboard
```
*配置修改后，重启 Zathura 生效。*
