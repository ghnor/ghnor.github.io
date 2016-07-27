---
title: 清空Activity堆栈
date: 2016-07-28 07:38:12
categories: Android
tags:
toc: true
---
```java
intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK);
```