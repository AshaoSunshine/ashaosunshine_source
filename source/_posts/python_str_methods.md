---
title: python中字符串一些方法
date: 2021-07-09 15:56:58
tags: python
categories: python
keywords: python, str
top: 
image: 
---
平时str相关方法用的少，正好看到，笔记一下。
<!--more-->

## Step1 str.center()

```python
str.center(w)
```
返回一个字符串，原字符串居中，使用空格填充新字符串，使其长度为w。


## Step2 str.count()

```python
str.count(item) 
```
返回item出现的次数。


## Step3 str.ljust()
```python
str.ljust(w)
```
返回一个字符串，将原字符串靠左放置并填充空格至长度w。


## Step4 str.rjust()
```python
str.rjust(w)
```
返回一个字符串，将原字符串靠右放置并填充空格至长度w。


## Step5 str.lower()
```python
str.lower()
```
返回均为小写字母的字符串。


## Step6 str.upper()
```python
str.upper()
```
返回均为大写字母的字符串。


## Step7 str.find()
```python
str.find(item)
```
返回item第一次出现时的下标。


## Step8 str.split()
```python
str.split(schar)
```
在schar位置将字符串分割成子串。  
split在处理数据的时候非常有用。split接受一个字符串，并且返回一个又分割字符作为分割点的字符串列表。
