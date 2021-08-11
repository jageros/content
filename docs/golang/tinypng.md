---
type: "posts"
author: "jager"
title: "批量图片压缩go实现"
date: "2021-08-03"
tags: [
"go",
"tinyPng",
]
draft: true
---
> 基于golang和tinypng的图片批量压缩工具实现

<!--more-->

## 准备工作
1. 首先通过[tinyPNG](https://tinypng.com/developers)获得一个api_key用于压缩图片api的验证，输入名称+邮箱，然后打开收到的邮件里的链接即可获得，一个key每月免费500次，即一个月可以压缩500张图片，500张对我个人使用够用了。
   ![](https://gitee.com/jayos/imgs/raw/master/20200413/202004131922152.png)
2. 然后创建一个码云的公共仓库用来存储压缩后的图片，gitee账号设置里面生成一个access_token用于提交图片。
   【PS：如果不用作图床，只需要把压缩后的图片存储到本地的话，可以不需要此步骤】
   ![](https://gitee.com/jayos/imgs/raw/master/20200413/202004131922071.png)

## 获取程序

## 配置tinyPNG以及gitee
1. 更改程序当前路径下的config.toml文件，具体看注释，根据需求选择是否需要填写gitee配置和输出文件夹。
   ![](https://gitee.com/jayos/imgs/raw/master/20200413/202004131945451.png)