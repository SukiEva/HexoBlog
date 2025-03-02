---
title: Linux指令之系统管理
toc: true
date: 2021-11-07 22:50
tags:
  - Linux
categories: [Programming Language,Shell]
updated: 2021-11-07 22:50
references:
  - title: 阿里云
    url: https://developer.aliyun.com/
---

Linux 中常用的系统工作命令以及系统状态检测命令。

<!-- more -->

## 常用系统工作命令

### Echo ：在终端输出字符串或变量提取后的值

命令描述：echo 命令用于在终端输出字符串或变量提取后的值。

命令格式：`echo [字符串 | $变量]`

命令用法示例：

**显示普通字符串**

```bash
echo "Hello World"
```

**显示变量**
首先在 shell 环境中定义一个临时变量 name，
使用 echo 命令将变量 name 的值显示到终端。

```bash
export name="Tom"
echo $name
```

**显示结果定向至文件**
以下命令会将文本 This is a test text.输出重定向到文件 test.txt 中，如果文件已存在，将会覆盖文件内容，如果不存在则创建。其中>符号表示输出重定向。

```bash
echo "This is a test text." > test.txt
```

如果您希望将文本追加到文件内容最后，而不是覆盖它，请使用>>输出追加重定向符号。

**显示命令执行结果**
以下命令将会在终端显示当前的工作路径。

```bash
echo `pwd`
```

注意：pwd 命令是用一对反引号（``）包裹，而不是一对单引号（''）。
使用 `$(command)` 形式可以达到相同效果。

```bash
echo $(pwd)
```

### Date ：显示和设置系统的时间和日期

命令描述：date 命令用于显示和设置系统的时间和日期。

命令格式：`date [选项] [+ 格式]`

其中，时间格式的部分控制字符解释如下：

<table class=""><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i19.25da70089Mos0m">字符</th><th>说明</th></tr></thead><tbody><tr><td data-spm-anchor-id="a2c6h.13858378.0.i18.25da70089Mos0m">%a</td><td>当地时间的星期名缩写（例如： 日，代表星期日）</td></tr><tr><td>%A</td><td>当地时间的星期名全称 （例如：星期日）</td></tr><tr><td>%b</td><td>当地时间的月名缩写 （例如：一，代表一月）</td></tr><tr><td>%B</td><td>当地时间的月名全称 （例如：一月）</td></tr><tr><td>%c</td><td>当地时间的日期和时间 （例如：2005 年 3 月 3 日 星期四 23:05:25）</td></tr><tr><td>%C</td><td>世纪；比如 %Y，通常为省略当前年份的后两位数字（例如：20）</td></tr><tr><td>%d</td><td>按月计的日期（例如：01）</td></tr><tr><td>%D</td><td>按月计的日期；等于%m/%d/%y</td></tr><tr><td>%F</td><td>完整日期格式，等价于 %Y-%m-%d</td></tr><tr><td>%j</td><td>按年计的日期（001-366）</td></tr><tr><td>%p</td><td>按年计的日期（001-366）</td></tr><tr><td>%r</td><td>当地时间下的 12 小时时钟时间 （例如：11:11:04 下午）</td></tr><tr><td>%R</td><td>24 小时时间的时和分，等价于 %H:%M</td></tr><tr><td>%s</td><td>自 UTC 时间 1970-01-01 00:00:00 以来所经过的秒数</td></tr><tr><td>%T</td><td>时间，等于%H:%M:%S</td></tr><tr><td>%U</td><td>一年中的第几周，以周日为每星期第一天（00-53）</td></tr><tr><td>%x</td><td>当地时间下的日期描述 （例如：12/31/99）</td></tr><tr><td>%X</td><td>当地时间下的时间描述 （例如：23:13:48）</td></tr><tr><td>%w</td><td>一星期中的第几日（0-6），0 代表周一</td></tr><tr><td>%W</td><td>一年中的第几周，以周一为每星期第一天（00-53）</td></tr></tbody></table>

命令用法示例：

**按照默认格式查看当前系统时间**

```bash
date
```

**按照指定格式查看当前系统时间**

```bash
date "+%Y-%m-%d %H:%M:%S"
```

**查看今天是当年中的第几天**

```bash
date "+%j"
```

**将系统的当前时间设置为 2020 年 02 月 20 日 20 点 20 分 20 秒**

```bash
date -s "20200220 20:20:20"
```

**校正系统时间，与网络时间同步**

a. 安装 ntp 校时工具

```bash
yum -y install ntp
```

b. 用 ntpdate 从时间服务器更新时间

```bash
ntpdate time.nist.gov
```

### Wget ：在终端中下载文件

命令描述：在终端中下载文件。

命令格式：`wget [参数] 下载地址 `

参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i24.25da70089Mos0m">参数</th><th>作用</th></tr></thead><tbody><tr><td>-b</td><td>后台下载</td></tr><tr><td>-P</td><td>下载到指定目录</td></tr><tr><td>-t</td><td>最大重试次数</td></tr><tr><td>-c</td><td>断点续传</td></tr><tr><td>-p</td><td>下载页面内所有资源，包括图片、视频等</td></tr><tr><td>-r</td><td>递归下载</td></tr></tbody></table>

命令使用示例：

**下载一张图片到路径/root/static/img/中，-p 参数默认值为当前路径，如果指定路径不存在会自动创建。**

```bash
wget -P /root/static/img/ http://img.alicdn.com/tfs/TB1.R._t7L0gK0jSZFxXXXWHVXa-2666-1500.png
```

### Ps ：查看系统中的进程状态

命令描述：ps 命令用于查看系统中的进程状态。

命令格式：`ps [参数]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i27.25da70089Mos0m">参数</th><th>作用</th></tr></thead><tbody><tr><td>-a</td><td>显示现行终端机下的所有程序，包括其他用户的程序</td></tr><tr><td data-spm-anchor-id="a2c6h.13858378.0.i26.25da70089Mos0m">-u</td><td>以用户为主的格式来显示程序状况</td></tr><tr><td>-x</td><td>显示没有控制终端的进程，同时显示各个命令的具体路径</td></tr><tr><td>-e</td><td>列出程序时，显示每个程序所使用的环境变量</td></tr><tr><td>-f</td><td>显示当前所有的进程</td></tr><tr><td>-t</td><td>指定终端机编号，并列出属于该终端机的程序的状况</td></tr></tbody></table>

命令使用示例：

```bash
ps -ef | grep sshd
```

### Top ：动态地监视进程活动与系统负载等信息

命令描述：top 命令动态地监视进程活动与系统负载等信息。

命令使用示例：

```bash
top
```

![top](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/top.png)

命令输出参数解释：

以上命令输出视图中分为两个区域，一个统计信息区，一个进程信息区。

- 统计信息区
	- 第一行信息依次为：系统时间、运行时间、登录终端数、系统负载（三个数值分别为 1 分钟、5 分钟、15 分钟内的平均值，数值越小意味着负载越低）。
	- 第二行信息依次为：进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。
	- 第三行信息依次为：用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。
	- 第四行信息依次为：物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。
	- 第五行信息依次为：虚拟内存总量、虚拟内存使用量、虚拟内存空闲量、预加载内存量。
- 进程信息区 <table> <caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i35.25da70089Mos0m">列名</th><th>含义</th></tr></thead><tbody><tr><td>PID</td><td>进程 ID</td></tr><tr><td>USER</td><td>进程所有者的用户名</td></tr><tr><td>PR</td><td>进程优先级</td></tr><tr><td>NI</td><td>nice 值。负值表示高优先级，正值表示低优先级</td></tr><tr><td>VIRT</td><td>进程使用的虚拟内存总量，单位 kb</td></tr><tr><td>RES</td><td>进程使用的、未被换出的物理内存大小，单位 kb</td></tr><tr><td>SHR</td><td data-spm-anchor-id="a2c6h.13858378.0.i34.25da70089Mos0m">共享内存大小，单位 kb</td></tr><tr><td>S</td><td>进程状态<ul><li>D：不可中断的睡眠状态</li><li>R：正在运行</li><li>S：睡眠</li><li>T：停止</li><li>Z：僵尸进程</li></ul></td></tr><tr><td>%CPU</td><td>上次更新到现在的 CPU 时间占用百分比</td></tr><tr><td>%MEM</td><td>进程使用的物理内存百分比</td></tr><tr><td>TIME+</td><td>进程使用的 CPU 时间总计，单位 1/100 秒</td></tr><tr><td>COMMAND</td><td>命令名</td></tr></tbody> </table>
	按 q 键退出监控页面。

### Pidof ：查询指定服务进程的 PID 值

命令描述：pidof 命令用于查询指定服务进程的 PID 值。

命令格式：`pidof [服务名称]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i38.25da70089Mos0m">参数</th><th>说明</th></tr></thead><tbody><tr><td>-s</td><td>仅返回一个进程号</td></tr><tr><td>-c</td><td>只显示运行在 root 目录下的进程，这个选项只对 root 用户有效</td></tr><tr><td>-o</td><td>忽略指定进程号的进程</td></tr><tr><td>-x</td><td>显示由脚本开启的进程</td></tr></tbody></table>

命令使用示例：

**查询出 crond 服务下的所有进程 ID。**

```bash
pidof crond
```

### Kill ：终止指定 PID 的服务进程

命令描述：kill 命令用于终止指定 PID 的服务进程。

kill 可将指定的信息送至程序。预设的信息为 SIGTERM(15)，可将指定程序终止。若仍无法终止该程序，可使用 SIGKILL(9) 信息尝试强制删除程序。

命令格式：`kill [参数] [进程 PID]`

命令使用示例：

**删除 pid 为 1247 的进程。**

```bash
kill -9 1247
```

### Killall ：终止指定名称的服务对应的全部进程

命令描述：killall 命令用于终止指定名称的服务对应的全部进程。

命令格式：`killall [进程名称]`

命令使用示例：

**删除 crond 服务下的所有进程。**

```bash
killall crond
```

### Reboot ：重启系统

命令描述：reboot 命令用来重启系统。

命令格式：`reboot [-n] [-w] [-d] [-f] [-i]`

命令参数说明：

- -n：保存数据后再重新启动系统。
- -w：仅做测试，并不是真的将系统重新开机，只会把重新开机的数据写入记录文件/var/log/wtmp。
- -d：重新启动时不把数据写入记录文件/var/tmp/wtmp。
- -f：强制重新开机，不调用 shutdown 指令的功能。
- -i：关闭网络设置之后再重新启动系统。

命令使用示例：

```bash
reboot
```

### Poweroff ：关闭系统

命令描述：poweroff 命令用来关闭系统。

命令使用示例：

```bash
poweroff
```

## 系统状态检测命令

### Ifconfig ：获取网卡配置与网络状态等信息

命令描述：ifconfig 命令用于获取网卡配置与网络状态等信息。

命令示例：

![ifconfig](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/ifconfig.png)

命令输出说明：

- 第一部分的第一行显示网卡状态信息。
	- eth0 表示第一块网卡。
	- UP 代表网卡开启状态。
	- RUNNING 代表网卡的网线被接上。
	- MULTICAST 表示支持组播。
- 第二行显示网卡的网络信息。
	- inet（IP 地址）：172.16.132.195。
	- broadcast（广播地址）：172.16.143.255。
	- netmask（掩码地址）：255.255.240.0。
	- RX 表示接收数据包的情况，TX 表示发送数据包的情况。
	- lo 表示主机的回环网卡，是一种特殊的网络接口，不与任何实际设备连接，而是完全由软件实现。与回环地址（127.0.0.0/8 或 ::1/128）不同，回环网卡对系统显示为一块硬件。任何发送到该网卡上的数据都将立刻被同一网卡接收到。

### Uname ：查看系统内核与系统版本等信息

命令描述：uname 命令用于查看系统内核与系统版本等信息。

命令语法：`uname [-amnrsv][--help][--version]`

命令使用示例：

**显示系统信息。**

```bash
uname -a
```

**显示当前系统的硬件架构。**

```bash
uname -i
```

**显示操作系统发行编号。**

```bash
uname -r
```

**显示操作系统名称。**

```bash
uname -s
```

**显示主机名称。**

```bash
uname -n
```

### Uptime ：查看系统的负载信息

命令描述：uptime 用于查看系统的负载信息。

命令使用示例：

![uptime](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/uptime.png)

命令输出说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i55.25da70089Mos0m">负载信息</th><th data-spm-anchor-id="a2c6h.13858378.0.i54.25da70089Mos0m">命令输出值</th></tr></thead><tbody><tr><td>当前服务器时间</td><td>14:20:27</td></tr><tr><td>当前服务器运行时长</td><td>2 min</td></tr><tr><td>当前用户数</td><td>2 users</td></tr><tr><td>当前负载情况</td><td><code>load average: 0.03, 0.04, 0.02</code>（分别取 1min，5min，15min 的均值）</td></tr></tbody></table>

### Free ：显示当前系统中内存的使用量信息

命令描述：free 用于显示当前系统中内存的使用量信息。

命令语法：`free [-bkmotV][-s <间隔秒数>]`

命令参数说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i59.25da70089Mos0m">参数</th><th>说明</th></tr></thead><tbody><tr><td>-b</td><td>以 Byte 为单位显示内存使用情况</td></tr><tr><td>-k</td><td>以 KB 为单位显示内存使用情况</td></tr><tr><td>-m</td><td>以 MB 为单位显示内存使用情况</td></tr><tr><td>-h</td><td>以合适的单位显示内存使用情况，最大为三位数，自动计算对应的单位值。</td></tr></tbody></table>

命令使用示例：

![free](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/free.png)

命令输出说明：

<table><caption></caption><colgroup><col><col></colgroup><thead><tr><th data-spm-anchor-id="a2c6h.13858378.0.i64.25da70089Mos0m">参数</th><th>说明</th></tr></thead><tbody><tr><td data-spm-anchor-id="a2c6h.13858378.0.i63.25da70089Mos0m">total</td><td>物理内存总数</td></tr><tr><td>used</td><td>已经使用的内存数</td></tr><tr><td>free</td><td>空间的内存数</td></tr><tr><td>share</td><td>多个进程共享的内存总额</td></tr><tr><td>buff/cache</td><td>应用使用内存数</td></tr><tr><td>available</td><td>可用的内存数</td></tr><tr><td>Swap</td><td>虚拟内存（阿里云 ECS 服务器默认不开启虚拟内存）</td></tr></tbody></table>

### Who ：显示关于当前在本地系统上的所有用户的信息

命令描述：who 命令显示关于当前在本地系统上的所有用户的信息。

命令使用示例：

**显示当前登录系统的用户**

```bash
who
```

**显示用户登录来源**

```bash
who -l -H
```

**只显示当前用户**

```bash
who -m -H
```

**精简模式显示**

```bash
who -q
```

### Last ：显示用户最近登录信息

命令描述： last 命令用于显示用户最近登录信息。

```bash
last
```

### History ：显示历史执行过的命令

命令描述：history 命令用于显示历史执行过的命令。

bash 默认记录 1000 条执行过的历史命令，被记录在~/.bash_history 文件中。

命令使用示例：

**显示最新 10 条执行过的命令。**

```bash
history 10
```

**清除历史记录。**

```bash
history -c
```
