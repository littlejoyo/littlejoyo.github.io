---
title: Linux：如何优雅的查看生产日志？
date: 2019-12-19 15:03:42
categories:
  - [服务器]
  - [Linux]
tag:
    - Linux
    - 日志
    - 服务器
---

- 我们都知道，生产环境下的日志迭代速度是非常快

- 倘若仅使用cat命令来查看日志文件，不但无法查看实时日志，严重的甚至还可能影响服务器的运行

- 本篇重点介绍：如何使用Linux命令优雅的查看日志，快速查询到想要的日志内容！

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/

> 个人博客：https://littlejoyo.github.io/

# 1.tail 命令

> tail 命令可以查看实时日志的更新，当日志有更新的时候，实时打印到控制台显示。

## 1.1 常用命令

- `tail -f online.log` : 把生产日志文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要文件更新就可以看到最新的文件内容。(停止显示：ctrl+c)

- `tail online.log` : 显示 online.log 文件的最后 10 行

- `tail -n 10 online.log` : 显示文件 online.log 的最后 10 行日志

- `tail -n +10 online.log` : 显示第10行之后的所有日志内容

- `tail -100f online.log` : 实时监控100行日志内容（不停刷新，保持100条日志）

- `tail -c 10 online.log` : 显示日志文件 online.log 的最后10个字符

## 1.2 tail 参数说明

|   参数    |         作用          |
| :-------: | :-------------------: |
|    -f     |       循环读取        |
|    -q     |    不显示处理信息     |
|    -v     |  显示详细的处理信息   |
| -c <行数> |     显示的字符数      |
| -n <行数> | 显示文件的尾部n行内容 |

# 2.head命令

> head命令的用法和tail命令相似，但是前者从头部开始扫描内容，后者是从尾部开始输出内容

## 2.1 常用命令

- `head online.log` : 显示 online.log 文件的从头开始的前10 行日志内容

- `head -n 10 online.log` : 显示文件 online.log 的从首行开始的 10 行日志

- `head -n +100 online.log` : 显示第100行之前的所有日志内容

- `head -c 10 online.log` : 显示日志文件 online.log 的最开始的10个字符

# 3.grep 命令

> Linux grep 命令用于查找文件里符合条件的字符串。

## 3.1 常用命令

- `grep '12:00:00' online.log` : 打印符合`12:00:00`匹配行的日志内容

- `grep -n '12:00:00' online.log ` : 打印符合`12:00:00`匹配行的日志内容并显示日志行数

- `grep -n -A 3 '12:00:00' online.log` : 打印匹配行的日志内容和之后3行的日志

- `grep -n -B 3 '12:00:00' online.log` : 打印匹配行的日志内容和之前3行的日志

- `grep -n -C 3 '12:00:00' online.log` : 打印匹配行的日志内容和前后各3行的日志

- `grep -e '正则表达式' online.log ` : 从online.log中查找与正则表达式匹配的行

- `grep –i "被查找的字符串" oneline.log` : 查找时不区分大小写

- `grep '123' online.log | grep '456'` : 显示既匹配 ‘123’又匹配 ‘456’的行

## 3.2 grep 参数说明

[grep命令和参数介绍](https://www.linuxcool.com/grep)

# 4.sed 命令

> Linux sed 命令是利用脚本来处理文本文件。此处只说明如何用来查看具体时间段的日志内容。

```markdown
sed -n '/起始时间/,/结束时间/p' 日志文件
例如：
sed -n '/2019-12-17 12:00:00/,/2019-12-17 12:10:10/p'  online.log
// 查看2019-12-17 12:00:00-12:10:10 时间段内的日志内容
```

[sed命令和参数介绍](https://www.linuxcool.com/sed)

# 5.less 命令 和 more 命令

> less和more都可以对读取文件内容并进行分页处理，但是两者最大的区别就是less更加的灵活。

## 5.1 more

- more功能类似 cat ，cat命令是整个文件的内容从上到下显示在屏幕上。 more会以一页一页的显示方便使用者逐页阅读。

- 让画面在显示满一页时暂停，此时可按空格健继续显示下一个画面，或按Q键停止显示。

- more不可以回去，就是不可以向前，只能向后，况且只能使用Enter和Space向后翻动。

- more会读取整个文件的内容进行分页处理，加载速度低于less

## 5.2 less

### 5.2.1 less 命令介绍

- less 工具也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。

- less与more最大的不同就是能够前后翻页，不必读取整个文件，加载速度比more更快。

### 5.2.2 less命令用法

- `less -N online.log` : 查看online.log文件并显示行号，支持前后翻页

- `less -N +10g online.log` : 定位到online.log文件的第10行开始显示内容

- less查看文件后，`/2019-12-19 10:00:00` : 自动高亮显示查找到的内容，后续n下一个，N上一个

- `cat -n online.log | grep 'debug' | less` : 分页显示过滤到的内容（less和more一般都会搭配管道符`|` 一起使用）

### 5.2.3 less命令参数

| 参数       | 详解                         |
| ---------- | ---------------------------- |
| d          | 向下翻页                     |
| u          | 向上翻页                     |
| g          | 跳到首行                     |
| G          | 跳到底部                     |
| ？查找内容 | 向上查找匹配的内容并高亮显示 |
| / 查找内容 | 向下查找匹配的内容并高亮显示 |
| n          | 下一个                       |
| N          | 上一个                       |
| q          | 退出less命令                 |

[less命令和参数介绍](https://www.linuxcool.com/less)

# 6.总结

- `tail` 、`grep`、`sed` 、`less` 等命令都可以实现日志的查看

- 不要再单纯使用`cat` 去查看日志文件，学会以上的命令灵活变通，根据需求选择合适的命令

- 大部分情况下，都会再加入管道符 `|` 进行多重条件的筛选，所以命令的应用会变得更加的灵活 

- 管道符 `|` 的作用 ：`命令1|命令2` （将命令1的输出内容作为命令2的输入）

  ```shell
  cat -n online.log | grep 'abc' | less
  ```

  