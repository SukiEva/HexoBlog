---
title: Linux指令之文本处理
toc: true
date: 2021-11-07 22:38
tags:
  - Linux
categories: [Programming Language,Shell]
updated: 2021-11-07 22:38
references:
  - title: 阿里云
    url: https://developer.aliyun.com/
---

如何使用 Linux 系统中的文本编辑工具 Vim 以及文本处理命令。

<!-- more -->

## 文本编辑工具 Vim

vim 的三种操作模式：

- 命令模式（Command mode）
- 输入模式（Insert mode）
- 底线命令模式（Last line mode）

三种模式切换快捷键：

| 模式     | 快捷键   |
| ------ | ----- |
| 命令模式   | ESC   |
| 输入模式   | i 或 a   |
| 底线命令模式 | **:** |

### 命令模式

在命令模式中控制光标移动和输入命令，可对文本进行复制、粘贴、删除和查找等工作。

使用命令 vim filename 后进入编辑器视图后，默认模式就是命令模式，此时敲击键盘字母会被识别为一个命令，例如在键盘上连续敲击两次 d，就会删除光标所在行。

以下是在命令模式中常用的快捷操作：

<table class="" style="width: 100%;" data-spm-anchor-id="a2c6h.13858378.0.i12.31941d111Dv71Z"><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i11.31941d111Dv71Z">操作</th><th>快捷键</th></tr></thead><tbody><tr><td>光标左移</td><td>h</td></tr><tr><td>光标右移</td><td>l（小写 L）</td></tr><tr><td data-spm-anchor-id="a2c6h.13858378.0.i13.31941d111Dv71Z">光标上移</td><td>k</td></tr><tr><td>光标下移</td><td>j</td></tr><tr><td>光标移动到下一个单词</td><td>w</td></tr><tr><td>光标移动到上一个单词</td><td>b</td></tr><tr><td>移动游标到第 n 行</td><td>nG</td></tr><tr><td>移动游标到第一行</td><td>gg</td></tr><tr><td>移动游标到最后一行</td><td>G</td></tr><tr><td>快速回到上一次光标所在位置</td><td>Ctrl+o</td></tr><tr><td>删除当前字符</td><td>x</td></tr><tr><td>删除前一个字符</td><td>X</td></tr><tr><td>删除整行</td><td>dd</td></tr><tr><td>删除一个单词</td><td>dw 或 daw</td></tr><tr><td>删除至行尾</td><td>d$或 D</td></tr><tr><td>删除至行首</td><td>d^</td></tr><tr><td>删除到文档末尾</td><td>dG</td></tr><tr><td>删除至文档首部</td><td>d1G</td></tr><tr><td>删除 n 行</td><td>ndd</td></tr><tr><td>删除 n 个连续字符</td><td>nx</td></tr><tr><td>将光标所在位置字母变成大写或小写</td><td>~</td></tr><tr><td>复制游标所在的整行</td><td>yy（3yy 表示复制 3 行）</td></tr><tr><td>粘贴至光标后（下）</td><td>p</td></tr><tr><td>粘贴至光标前（上）</td><td>P</td></tr><tr><td>剪切</td><td>dd</td></tr><tr><td>交换上下行</td><td>ddp</td></tr><tr><td>替换整行，即删除游标所在行并进入插入模式</td><td>cc</td></tr><tr><td>撤销一次或 n 次操作</td><td>u{n}</td></tr><tr><td>撤销当前行的所有修改</td><td>U</td></tr><tr><td>恢复撤销操作</td><td>Ctrl+r</td></tr><tr><td>整行将向右缩进</td><td>&gt;&gt;</td></tr><tr><td>整行将向左退回</td><td>&lt;&lt;</td></tr><tr><td>若档案没有更动，则不储存离开，若档案已经被更动过，则储存后离开</td><td>ZZ</td></tr></tbody></table>

### 输入模式

在命令模式下按 i 或 a 键就进入了输入模式，在输入模式下，您可以正常的使用键盘按键对文本进行插入和删除等操作。

### 底线命令模式

在命令模式下按: 键就进入了底线命令模式，在底线命令模式中可以输入单个或多个字符的命令。

以下是底线命令模式中常用的快捷操作：

<table class="" style="width: 100%;"><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i19.31941d111Dv71Z">操作</th><th>命令</th></tr></thead><tbody><tr><td>保存</td><td>:w</td></tr><tr><td>退出</td><td>:q</td></tr><tr><td>保存并退出</td><td>:wq（<code>:wq!</code>表示强制保存退出）</td></tr><tr><td>将文件另存为其他文件名</td><td>:w new_filename</td></tr><tr><td>显示行号</td><td>:set nu</td></tr><tr><td>取消行号</td><td>:set nonu</td></tr><tr><td data-spm-anchor-id="a2c6h.13858378.0.i18.31941d111Dv71Z">使本行内容居中</td><td>:ce</td></tr><tr><td>使本行文本靠右</td><td>:ri</td></tr><tr><td>使本行内容靠左</td><td>:le</td></tr><tr><td>向光标之下寻找一个名称为 word 的字符串</td><td>:/word</td></tr><tr><td>向光标之上寻找一个字符串名称为 word 的字符串</td><td>:?word</td></tr><tr><td>重复前一个搜寻的动作</td><td>:n</td></tr><tr><td>从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2</td><td><code>:1,$s/word1/word2/g</code> 或 <code><span>&nbsp;</span>:%s/word1/word2/g</code></td></tr></tbody></table>

## 文本文件查看命令

### Cat ：查看内容较少的纯文本文件

命令描述：cat 命令用于查看内容较少的纯文本文件。

命令格式：`cat [选项] [文件]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i26.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-n 或 --number</td><td>显示行号</td></tr><tr><td>-b 或 --number-nonblank</td><td>显示行号，但是不对空白行进行编号</td></tr><tr><td data-spm-anchor-id="a2c6h.13858378.0.i25.31941d111Dv71Z">-s 或 --squeeze-blank</td><td>当遇到有连续两行以上的空白行，只显示一行的空白行</td></tr></tbody></table>

### More ：从前向后分页显示文件内容

命令描述：more 命令从前向后分页显示文件内容。

常用操作命令：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i31.31941d111Dv71Z">操作</th><th>作用</th></tr></thead><tbody><tr><td>Enter</td><td>向下 n 行，n 需要定义，默认为 1 行</td></tr><tr><td>Ctrl+F 或空格键（Space）</td><td>向下滚动一页</td></tr><tr><td>Ctrl+B</td><td>向上滚动一页</td></tr><tr><td>=</td><td>输出当前行的行号</td></tr><tr><td>!命令</td><td>调用 Shell 执行命令</td></tr><tr><td>q</td><td>退出 more</td></tr></tbody></table>

### Less ：对文件或其它输出进行分页显示

命令描述：less 命令可以对文件或其它输出进行分页显示，与 moe 命令相似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动。

命令格式：`less [参数] 文件 `

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i33.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-e</td><td>当文件显示结束后，自动离开</td></tr><tr><td>-m</td><td data-spm-anchor-id="a2c6h.13858378.0.i32.31941d111Dv71Z">显示类似more 命令的百分比</td></tr><tr><td>-N</td><td>显示每行的行号</td></tr><tr><td>-s</td><td>显示连续空行为一行</td></tr></tbody></table>

命令常用操作：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th>快捷键</th><th>说明</th></tr></thead><tbody><tr><td data-spm-anchor-id="a2c6h.13858378.0.i34.31941d111Dv71Z">/字符串</td><td>向下搜索字符串</td></tr><tr><td>?字符串</td><td>向上搜索字符串</td></tr><tr><td>n</td><td>重复前一个搜索</td></tr><tr><td>N</td><td>反向重复前一个搜索</td></tr><tr><td>b 或<code>pageup</code>键</td><td>向上翻一页</td></tr><tr><td>空格键或<code>pagedown</code>键</td><td>向下翻一页</td></tr><tr><td>u</td><td>向前翻半页</td></tr><tr><td>d</td><td>向后翻半页</td></tr><tr><td>y</td><td>向前滚动一行</td></tr><tr><td>回车键</td><td>向后滚动一行</td></tr><tr><td>q</td><td>退出less 命令</td></tr></tbody></table>

命令使用示例：

查看命令历史使用记录并通过less 分页显示。

```bash
history | less
```

### Head ：查看文件开头指定行数的内容

命令描述：head 命令用于查看文件开头指定行数的内容。

命令格式：`head [参数] [文件]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i38.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-n [行数]</td><td>显示开头指定行的文件内容，默认为 10</td></tr><tr><td>-c [字符数]</td><td>显示开头指定个数的字符数</td></tr><tr><td>-q</td><td>不显示文件名字信息，适用于多个文件，多文件时默认会显示文件名</td></tr></tbody></table>

命令使用示例：

查看/etc/passwd 文件的前 5 行内容。

```bash
head -5 /etc/passwd
```

### Tail ：查看文档的后 N 行或持续刷新内容

命令描述：tail 命令用于查看文档的后 N 行或持续刷新内容。

命令格式：`tail [参数] [文件]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i41.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-f</td><td>显示文件最新追加的内容</td></tr><tr><td>-q</td><td>当有多个文件参数时，不输出各个文件名</td></tr><tr><td>-v</td><td>当有多个文件参数时，总是输出各个文件名</td></tr><tr><td>-c [字节数]</td><td>显示文件的尾部 n 个字节内容</td></tr><tr><td>-n [行数]</td><td>显示文件的尾部 n 行内容</td></tr></tbody></table>

命令使用示例：

查看/var/log/messages 系统日志文件的最新 10 行，并保持实时刷新。

```bash
tail -f -n 10 /var/log/messages
```

### Stat ：显示文件的详细信息

命令描述：用来显示文件的详细信息，包括 inode、atime、mtime、ctime 等。

命令使用示例：

查看/etc/passwd 文件的详细信息。

```bash
stat /etc/passwd
```

### Wc ：统计指定文本的行数、字数、字节数

命令描述：wc 命令用于统计指定文本的行数、字数、字节数。

命令格式：`wc [参数] [文件]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i46.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-l</td><td>只显示行数</td></tr><tr><td>-w</td><td>只显示单词数</td></tr><tr><td>-c</td><td>只显示字节数</td></tr></tbody></table>

命令使用示例：

统计/etc/passwd 文件的行数。

```bash
wc -l /etc/passwd
```

### File ：辨识文件类型

命令描述： file 命令用于辨识文件类型。

命令格式：`file [参数] [文件]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i48.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td data-spm-anchor-id="a2c6h.13858378.0.i47.31941d111Dv71Z">-b</td><td>列出辨识结果时，不显示文件名称</td></tr><tr><td>-c</td><td>详细显示指令执行过程，便于排错或分析程序执行的情形</td></tr><tr><td>-f [文件]</td><td>指定名称文件，其内容有一个或多个文件名称时，让 file 依序辨识这些文件，格式为每列一个文件名称</td></tr><tr><td>-L</td><td>直接显示符号连接所指向的文件类别</td></tr></tbody></table>

命令使用示例：

查看/var/log/messages 文件的文件类型。

```bash
file /var/log/messages
```

### Diff ：比较文件的差异

命令描述：diff 命令用于比较文件的差异。

命令格式：`diff [文件] [文件]`

## 文本处理命令

### Grep ：查找文件里符合条件的字符串

命令描述：grep 命令用于查找文件里符合条件的字符串。

grep 全称是 Global Regular Expression Print，表示全局正则表达式版本，它能使用正则表达式搜索文本，并把匹配的行打印出来。

在 Shell 脚本中，grep 通过返回一个状态值来表示搜索的状态：

- 0：匹配成功。
- 1：匹配失败。
- 2：搜索的文件不存在。

命令格式：`grep [参数] [正则表达式] [文件]`

命令常用参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i54.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-c 或 --count</td><td>计算符合样式的列数</td></tr><tr><td>-d recurse 或 -r</td><td>指定要查找的是目录而非文件</td></tr><tr><td>-e [范本样式]</td><td>指定字符串做为查找文件内容的样式</td></tr><tr><td>-E 或 --extended-regexp</td><td>将样式为延伸的正则表达式来使用</td></tr><tr><td>-F 或 --fixed-regexp</td><td>将样式视为固定字符串的列表</td></tr><tr><td>-G 或 --basic-regexp</td><td>将样式视为普通的表示法来使用</td></tr><tr><td>-i 或 --ignore-case</td><td>忽略字符大小写的差别</td></tr><tr><td>-n 或 --line-number</td><td>在显示符合样式的那一行之前，标示出该行的列数编号</td></tr><tr><td>-v 或 --revert-match</td><td>显示不包含匹配文本的所有行</td></tr></tbody></table>

命令使用示例：

查看 sshd 服务配置文件中监听端口配置所在行编号。

```bash
grep -n Port /etc/ssh/ssh_config
```

查询字符串在文本中出现的行数。

```bash
grep -c localhost /etc/hosts
```

反向查找，不显示符合条件的行。

```bash
ps -ef | grep sshd
ps -ef | grep -v grep | grep sshd
```

以递归的方式查找目录下含有关键字的文件。

```bash
grep -r *.sh /etc
```

使用正则表达式匹配 httpd 配置文件中异常状态码响应的相关配置。

```bash
grep 'ntp[0-9].aliyun.com' /etc/ntp.conf
```

### Sed ：文本处理

命令描述：sed 是一种流编辑器，它是文本处理中非常中的工具，能够完美的配合正则表达式使用。

1. 处理时，把当前处理的行存储在临时缓冲区中，称为模式空间（pattern space）。
2. 接着用 sed 命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。
3. 接着处理下一行，这样不断重复，直到文件末尾。

注意：

sed 命令不会修改原文件，例如删除命令只表示某些行不打印输出，而不是从原文件中删去。

如果要改变源文件，需要使用 -i 选项。
命令格式：`sed [参数] [动作] [文件]`

参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i63.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-e [script]</td><td>执行多个 script</td></tr><tr><td>-f [script 文件]</td><td>执行指定 script 文件</td></tr><tr><td>-n</td><td>仅显示 script 处理后的结果</td></tr><tr><td>-i</td><td>输出到原文件，静默执行（修改原文件）</td></tr></tbody></table>

动作说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i65.31941d111Dv71Z">动作</th><th>说明</th></tr></thead><tbody><tr><td>a</td><td>在行后面增加内容</td></tr><tr><td>c</td><td>替换行</td></tr><tr><td>d</td><td>删除行</td></tr><tr><td>i</td><td>在行前面插入</td></tr><tr><td>p</td><td>打印相关的行</td></tr><tr><td>s</td><td>替换内容</td></tr></tbody></table>

命令使用示例：

删除第 3 行到最后一行内容。

```bash
sed '3,$d' /etc/passwd
```

在最后一行新增行。

```bash
sed '$a admin:x:1000:1000:admin:/home/admin:/bin/bash' /etc/passwd
```

替换内容。

```bash
sed 's/SELINUX=disabled/SELINUX=enforcing/' /etc/selinux/config
```

替换行。

```bash
sed '1c abcdefg' /etc/passwd
```

### Awk ：文本处理

命令描述：和 sed 命令类似，awk 命令也是逐行扫描文件（从第 1 行到最后一行），寻找含有目标文本的行，如果匹配成功，则会在该行上执行用户想要的操作；反之，则不对行做任何处理。

命令格式：`awk [参数] [脚本] [文件]`

参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i72.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-F fs</td><td>指定以 fs 作为输入行的分隔符，awk 命令默认分隔符为空格或制表符</td></tr><tr><td data-spm-anchor-id="a2c6h.13858378.0.i71.31941d111Dv71Z">-f file</td><td>读取 awk 脚本</td></tr><tr><td>-v val=val</td><td>在执行处理过程之前，设置一个变量 var，并给其设置初始值为 val</td></tr></tbody></table>

内置变量：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i74.31941d111Dv71Z">变量</th><th>用途</th></tr></thead><tbody><tr><td>FS</td><td>字段分隔符</td></tr><tr><td>$n</td><td>指定分隔的第 n 个字段，如 $1、$3 分别表示第 1、第三列</td></tr><tr><td>$0</td><td>当前读入的整行文本内容</td></tr><tr><td>NF</td><td>记录当前处理行的字段个数（列数）</td></tr><tr><td>NR</td><td>记录当前已读入的行数</td></tr><tr><td>FNR</td><td>当前行在源文件中的行号</td></tr></tbody></table>

awk 中还可以指定脚本命令的运行时机。默认情况下，awk 会从输入中读取一行文本，然后针对该行的数据执行程序脚本，但有时可能需要在处理数据前运行一些脚本命令，这就需要使用 BEGIN 关键字，BEGIN 会在 awsk 读取数据前强制执行该关键字后指定的脚本命令。

和 BEGIN 关键字相对应，END 关键字允许我们指定一些脚本命令，awk 会在读完数据后执行它们。

命令使用示例：

查看本机 IP 地址。

```bash
ifconfig eth0 |awk '/inet/{print $2}'
```

查看本机剩余磁盘容量。

```bash
df -h |awk '/\/$/{print $4}'
```

统计系统用户个数。

```bash
awk -F: '$3<1000{x++} END{print x}' /etc/passwd
```

输出其中登录 Shell 不以 nologin 结尾（对第 7 个字段做!~反向匹配）的用户名、登录 Shell 信息。

```bash
awk -F: '$7!~/nologin$/{print $1,$7}' /etc/passwd
```

输出/etc/passwd 文件中前三行记录的用户名和用户 uid。

```bash
head -3 /etc/passwd | awk  'BEGIN{FS=":";print "name\tuid"}{print $1,"\t"$3}END{print "sum lines "NR}'
```

查看 tcp 连接数。

```bash
netstat -na | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

关闭指定服务的所有的进程。

```bash
ps -ef | grep httpd | awk {'print $2'} | xargs kill -9
```

### Cut ：切割字符串

命令描述：cut 命令主要用来切割字符串，可以对输入的数据进行切割然后输出。

命令格式：`cut [参数] [文件]`

参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i81.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-b</td><td>以字节为单位进行分割</td></tr><tr><td>-c</td><td>以字符为单位进行分割</td></tr><tr><td>-d</td><td>自定义分隔符，默认为制表符</td></tr></tbody></table>

### Tr ：对来自标准输入的字符进行替换、压缩和删除

命令描述：tr 命令用于对来自标准输入的字符进行替换、压缩和删除。

命令格式：`tr [参数] [文本]`

参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i84.31941d111Dv71Z">参数</th><th>说明</th></tr></thead><tbody><tr><td>-c</td><td>反选指定字符</td></tr><tr><td>-d</td><td>删除指定字符</td></tr><tr><td>-s</td><td>将重复的字符缩减成一个字符</td></tr><tr><td>-t [第一字符集] [第二字符集]</td><td>删除第一字符集较第二字符集多出的字符，使两个字符集长度相等</td></tr></tbody></table>

命令使用示例：

将输入字符由大写转换为小写。

```bash
echo "HELLO WORLD" | tr 'A-Z' 'a-z'
```

删除字符。

```bash
echo "hello 123 world 456" | tr -d '0-9'
```

压缩字符。

```bash
echo "thissss is      a text linnnnnnne." | tr -s ' sn'
```

产生随机密码。

```bash
cat /dev/urandom | tr -dc a-zA-Z0-9 | head -c 13
```
