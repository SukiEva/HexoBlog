---
title: Shell
tags:
  - Shell
categories: [Programming Language,Shell]
toc: true
date: 2021-12-28 20:14
updated: 2021-12-28 20:14
---

Shell 脚本的一些注意点

<!-- more -->

## 文件测试

```bash
if [ -e file/dir ] ## 如果文件/目录存在  
if [ -f file ] ## 如果文件存在  
if [ -d dir ]  ## 如果目录存在  
if [ -s file ] ## 如果文件存在且非空  
if [ -r file ] ## 如果文件存在且可读  
if [ -w file ] ## 如果文件存在且可写  
if [ -x file ] ## 如果文件存在且可执行
```

## 整数比较

```bash
if [ int1 -eq int2 ] ## 如果 ==  
if [ int1 -ne int2 ] ## 如果 !=  
if [ int1 -ge int2 ] ## 如果 >=  
if [ int1 -gt int2 ] ## 如果 >  
if [ int1 -le int2 ] ## 如果 <=  
if [ int1 -lt int2 ] ## 如果 <
```

## 字符串比较

```bash
if [ $string1 == $string2 ] ## 如果 == (字符串允许使用赋值号做等号)  
if [ $string1 != $string2 ] ## 如果 !=  
if [ -n $string ]           ## 如果string 长度非0 
if [ -z $string ]           ## 如果string 长度为0  
```
