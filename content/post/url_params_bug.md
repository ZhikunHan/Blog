---
title: "Url_params_bug"
date: 2022-09-17T10:51:04+08:00
math: true
slug: "Url Params Bug"
# weight: 1
# aliases: ["/first"]
tags: ["Bug","Coding"]
categories: ["Bug"]
# author: ["Me", "You"] # multiple authors
# showToc: true
# TocOpen: false
# draft: false
# hidemeta: false
# comments: false
# description: "Desc Text."
image: "https://api.ixiaowai.cn/api/api.php"
---

## 问题描述

在调试客户端向服务端传递参数时，url中的参数值出现+, 空格, \, ?, %, #, &等特殊符号的时候会自动变成空格，在服务器端无法获得正确的参数值。

### 解决方法

修改客户端，将客户端带“+”的参数值进行转义，将“+”转换成“%2B”，就能得到“+”了。
