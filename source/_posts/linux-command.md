---
title: 记录一些不是很常用或容易混淆的linux指令，包括快捷键和shell命令。
date: 2019-04-25 14:20:01
updated: 2019-04-25 14:20:01
tags: linux
---

## 命令编辑及光标移动
删除从开头到光标处的命令文本  
`ctrl + u`  
删除从光标到结尾处的命令文本  
`ctrl + k`
移动光标到命令开头  
`ctrl + a`
移动光标到命令结尾  
`ctrl + e`  
向前移动一个单词  
`alt + f`  
向后移动一个单词  
`alt + b`  
删除一个单词  
`ctrl + w`  
<!-- more -->
## 历史命令
查看历史命令  
`history` 
快速执行历史命令  
`! + 历史命令前序号`  
搜索历史命令  
`ctrl + r`  

## 实时查看日志
`tail -f example.log`  

## 进程
根据名称查找进程
`pgrep + name`  
根据名称杀死进程  
`killall + name`  
`pkill + name`  
查看进程运行时间  
`ps -p pid -o lstart,etime`  
中止一个进程(发送 SIGINT 信号给前台进程组中的所有进程)
`ctrl + c`  
挂起一个进程(发送 SIGTSTP 信号给前台进程组中的所有进程)  
`ctrl + z`

> ctrl-d 不是发送信号，而是表示一个特殊的二进制值，表示 EOF。  

恢复一个被挂起的进程至前台
`fg + [jobnumber]`  
恢复一个被挂起的进程至后台  
`bg + [jobnumber]`  
查看jobnumber  
`jobs`  

## 日志
记录日志文件同时输出到控制台  
`./example.sh | tee example.log`





