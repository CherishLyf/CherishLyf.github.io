---
layout:     post
title:      "ReactNative 初试 --- 环境搭建"
subtitle:   "搭配环境时遇到的坑"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - ReactNative
---

# 环境搭建

按照[官网](http://reactnative.cn/docs/0.39/getting-started.html#content)一步一步搭建环境，[常见问题汇总](http://bbs.reactnative.cn/topic/130/%E6%96%B0%E6%89%8B%E6%8F%90%E9%97%AE%E5%89%8D%E5%85%88%E6%9D%A5%E8%BF%99%E9%87%8C%E7%9C%8B%E7%9C%8B-react-native%E7%9A%84%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)。

自己实际搭建遇到的问题，

1.运行react-native run-android时报错，报错信息中含有No connected devices!字样

没有链接真机或者模拟器，连上真机解决。

2.红屏报错 `could not get batchedbridge, make sure your bundle is packaged correctly`

命令行输入 `react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/`，这时候还报错，没有这个路径，找到 `/app/src/main` 创建一个 `assets` 的目录，解决~~~
