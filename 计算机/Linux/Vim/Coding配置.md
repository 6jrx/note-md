### 编码格式配置

<br/>

修改配置文件：`vim ~/.vimrc`

`"`符号在这个配置文件中是注释符号（vimscript语法）

```text
" 显示行号
set number
" 自动缩进
set autoindent
" 启用语法高亮
syntax on

" 将 Tab 转换为 4 个空格
set tabstop=4     " 一个 Tab 显示为 4 个空格宽度
set shiftwidth=4  " 自动缩进时使用的空格数（如按 >> 或 << 时的缩进量）
set softtabstop=4 " 按退格键（Backspace）时删除 4 个空格（模拟 Tab 行为）
set expandtab     " 将 Tab 自动转换为空格（关键配置！）
```

<br/>

<br/>

**如果希望仅对 C 语言文件生效，在 .vimrc 中添加：**

<br/>

```text
autocmd FileType c setlocal tabstop=4 shiftwidth=4 softtabstop=4 expandtab

```

<br/>

**若某些场景需要保留 Tab 字符（如 Makefile），可禁用 expandtab：**

```text
autocmd FileType make setlocal noexpandtab

```
