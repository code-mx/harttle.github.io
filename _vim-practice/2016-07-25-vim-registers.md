---
title: 使用 Vim 寄存器
tags: Linux Vim Windows X11 宏 剪切板 寄存器 正则表达式
---

常见文本编辑器都会提供剪切板来支持复制粘贴，Vim也不例外。
不同的是Vim提供了10类共48个寄存器，提供无与伦比的寄存功能。
最常用的`y`操作将会拷贝到默认的匿名寄存器中，我们也可以指定具体拷贝到哪个寄存器中。

一般来讲，可以用`"{register}y`来拷贝到`{register}`中，
用`"{register}p`来粘贴`{register}`中的内容。例如：
`"ayy`可以拷贝当前行到寄存器`a`中，而`"ap`则可以粘贴寄存器`a`中的内容。

<!--more-->

除了`a-z`26个命名寄存器，Vim还提供了很多特殊寄存器。合理地使用可以极大地提高效率。例如：

* `"+p`可以粘贴剪切板的内容，
* `":p`可以粘贴上一个Vim命令（比如你刚刚费力拼写的正则表达式），
* `"/p`可以粘贴上一次搜索关键词（你猜的没错，正是normal模式下的`/foo`搜索命令）。

在Vim中可通过`:reg`来查看每个寄存器当前的值。

# 寄存器分类

Vim提供了10类寄存器，可在Vim中通过`:help registers`查看帮助。

1. 匿名寄存器 `""`
2. 编号寄存器 `"0` 到 `"9`
3. 小删除寄存器 `"-`
4. 26个命名寄存器 `"a` 到 `"z`
5. 3个只读寄存器 `":`, `".`, `"%`
6. Buffer交替文件寄存器 `"#`
7. 表达式寄存器 `"=`
8. 选区和拖放寄存器 `"*`, `"+`, `"~` 
9. 黑洞寄存器 `"_`
10. 搜索模式寄存器 `"/`

# 1. 匿名寄存器

使用`d`, `c`, `s`, `x`等会删除字符的命令时，被删除字符会进入匿名寄存器`""`。
你可以认为`""`寄存器是一个指针，指向刚才被存到的寄存器。

在[如何用Vim搭建IDE？][vim-ide]一文中提到，Mac下可通过下列设置来让Vim共享系统剪切板，
就是这个原理：所有删除和拷贝操作默认都会到匿名寄存器。

```vim
set clipboard=unnamed
```

使用`y`命令未指定寄存器会存到`"0`寄存器中，同时`""`会与该寄存器保有同样的值。
这意味着你使用`p`和`"p`总会得到同样的结果。

# 2. 编号寄存器

编号寄存器从`"0`到`"9`共10个，其中`"0`保存着拷贝来的字符串，`"1`到`"9`保存着删除掉的字符串。
删除操作符包括`s`, `c`, `d`, `x`。
删除掉的字符串会被存到`"1`中，上次删除的则会被存到`"2`中。以此类推，Vim会保存你最近的9次删除。

* 只有整行整行的删除，和通过段落级别的移动指令（包括``%,(,),/,`,?,n,N,{,}``）
  的删除才会被放到`"1`中。
* 当用户指定拷贝操作的寄存器时（如`"ap`），`"0`不会被写入；但删除操作一定会被写入到`"1`中。

> `"0`寄存器很有用，比如我们copy了一段文本然后用它替换另一段文本。
> 这时默认寄存器`""`中的值就变成了被替换文本，如果还需要用copy的文本继续替换的话就需要`"0p`了。

# 3. 小删除寄存器

不足一行的小删除则会被放到小删除寄存器中（`"-`），起作用的删除操作符也包括`s`, `c`, `d`, `x`。
例如：

```bash
dw    # 删除一个词
d9l   # 删除9个字符
cb    # 向前更改一个词
```

与`"0`寄存器类似，当用户指定寄存器并进行删除时，`"-`不会被写入。

# 4. 命名寄存器

命名寄存器有`"a`到`"z`共26个，这些寄存器只有当我们指定时才会被使用。
其实我们在录制宏时，所有键盘操作会以字符串的形式存到寄存器中。
例如录制一个宏存到`"a`寄存器中，内容为更改当前行`cc`，改为`foo`字符串：

```
qaccfoo
```

然后执行`:reg`来查看寄存器，可以发现`a`寄存器的值是`ccfoo`。

> 小技巧：当使用小写字母进行操作时会覆盖当前寄存器内容，当使用大写字母进行操作时，会追加当前寄存器内容。

# 5. 只读寄存器

只读寄存器共3个，它们的值是由Vim提供的，不允许改变：

* `".`：上次insert模式中插入的字符串。还记得吗？`.`命令可以重复上次操作，而`".`存储了上次插入。
* `"%`：当前文件名，不是全路径，也不是纯文件名，而是从当前Vim的工作目录到该文件的路径。例如此时Harttle的Vim中，`"%p`的结果为`_drafts/vim-registers.md`。
* `":`：上次命令模式下键入的命令。正如`@a`可以执行`"a`寄存器中的宏一样，`":`可以执行上次命令。

# 6. 交替文件寄存器

交替文件寄存器`"#`存储着当前Vim窗口（Window）的交替文件。**交替文件**（alternate file）是指
[Buffer][vim-buffer]中的上一个文件，可通过`Ctrl+^`来切换交替文件与当前文件。

> Window和Buffer有什么区别？参见[Vim 多文件编辑：窗口][vim-window]一文。

# 7. 表达式寄存器

表达式寄存器`"=`主要用于计算Vim脚本的返回值，并插入到文本中。
当我们键入`"=`后光标会移动到命令行，此时我们可以输入任何Vim脚本的表达式。
例如`3+2`，按下回车并且`p`则会得到`5`。

这在我们调试Vim脚本时非常有用，比如调用一个函数看它是否有正确的返回值。

# 8. 选择和拖放寄存器

选择和拖放寄存器包括`"*`, `"+`, 和`"~`，这三个寄存器的行为是和GUI相关的。

`"*`和`"+`在Mac和Windows中，都是指系统剪切板（clipboard），例如`"*yy`即可复制当前行到剪切板。
以供其他程序中粘贴。其他程序中复制的内容也会被存储到这两个寄存器中。
在X11系统中（绝大多数带有桌面环境的Linux发行版），二者是有区别的：

* `"*`指X11中的PRIMARY选区，即鼠标选中区域。在桌面系统中可按鼠标中键粘贴。
* `"+`指X11中的CLIPBOARD选区，即系统剪切板。在桌面系统中可按Ctrl+V粘贴。

> 上文所述的Mac下`set clipboard=unnamed`会使得系统剪切板寄存器`"*`和Vim默认的匿名寄存器`""`始终保有同样的值，即Vim和系统共用剪切板。

有文本拖拽到Vim时，被拖拽的文本被存储在`"~`中。Vim默认的行为是将`"~`中内容插入到光标所在位置。
当然你可以给`<DROP>`做键盘映射。

# 9. 黑洞寄存器

黑洞寄存器`"_`，所有删除或拷贝到黑洞寄存器的文本将会消失。
这是为了在删除文本的同时不影响任何寄存器的值，`"_`通常用于Vim脚本中。

# 10. 搜索寄存器

搜索寄存器`"/`用于存储上一次搜索的关键词。Vim中如何进行搜索呢？
在normal模式下按下`/`即进入search模式，输入关键字并按下回车即可。

该寄存器是可写的，例如`:let @/ = "harttle"`将会把`"harttle"`写入该寄存器。
下次搜索时不输入搜索词直接回车便会搜索`"harttle"`。

# 命令行模式拷贝

值得一提的时，任何寄存器中的值都是可以拷贝到命令模式下的。

比如对于寄存器`"a`中的值，在normal模式下可以通过`"ap`来粘贴；在command-line模式下通过`<Ctrl-R>a`来粘贴。这一操作存在风险，因为寄存器中的值可能是从网页中拷贝来的。

如果寄存器中的字符串存在`<Esc>`字符或`<CR>`字符，则会时Vim回到normal模式，
并继续执行寄存器中的命令。为了防范*剪切板劫持*，可以添加下列的Vim配置：

```vim
inoremap <C-r>+ <C-g>u<C-\><C-o>"+gP
```

> 该命令的解释请移步：<http://vim.wikia.com/wiki/Pasting_registers>

# 扩展阅读

* 剪切板与X11选区：<http://stackoverflow.com/questions/11489428/how-to-make-vim-paste-from-and-copy-to-systems-clipboard>
* Vikia-Pasting Registers: <http://vim.wikia.com/wiki/Pasting_registers>
* Vim Help: `:help registers`, `:help quotestar`, `:help quoteplus`

[vim-ide]: /2015/11/04/vim-ide.html
[vim-window]: /2015/11/14/vim-window.html
[vim-buffer]: /2015/11/17/vim-buffer.md