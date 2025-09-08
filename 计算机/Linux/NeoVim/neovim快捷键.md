*<u>基础操作可复用vim操作</u>*

<br/>

#### 0x01、多光标操作

需插件支持如 `vim-visual-multi`

`CTRL+n`：选中相同单词

<br/>

#### 0x02、符号跳转

依赖 `LSP` 或其他插件

`gd`：跳转到定义（Go to Definition）

`gr`：查看引用（Go to References）

`K`：显示符号文档（Hover Documentation）

<br/>

#### 0x03、模糊搜索

需插件如 `Telescope.nvim`

空格是插件定义的唤起符，官方叫`<leader>`键

` ff`：搜索文件

` fg`：全局搜索文本

` fb`：搜索缓冲区（Buffer）

<br/>

#### 0x04、窗口操作

大部分从vim中继承过来的

- 分屏

`:vsplit`：垂直分屏

`:split`：水平分屏

`CTRL+w+j`：切换分屏，`j`在这里是位移，可以使用其他位移键

`CTRL+w+=`：均衡分屏尺寸

<br/>

- 标签页

`:tabnew`：新建标签页

`gt`：切换到下一个标签页

`gT`：切换到上一个标签页

<br/>

#### 0x05、插件操作

空格表示插件的唤起符号，空格是插件默认配置唤起符号

- LazyVim快捷键

` l`：打开LazyVim插件管理界面

` e`：打开文件树

` tt`：打开终端

<br/>

- LSP功能

` ca`：代码操作

` rn`：重命名符号

` fd`：格式化文档

<br/>

- 调试

`]d`：跳转到下一个诊断

`[d`：跳转到上一个诊断

` xx`：打开问题面板（Trouble.nvim）

<br/>

#### 0x06、代码块操作

`zc`：折叠代码块，`zo`展开

`zR`：展开所有折叠，`zM`折叠所有