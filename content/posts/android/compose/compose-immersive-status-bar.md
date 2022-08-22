---
categories: ["Android", "Jetpack", "Compose"]
date: "2022-08-08"
description: "Compose处理沉浸式状态栏和底部安全区"
title: "Compose处理沉浸式状态栏和底部安全区"
type: "post"
---

## 概述
Compose官方示例代码中，细心的小伙伴会发现顶部系统状态栏背景颜色，默认情况下都是深蓝色，而状态栏的文字颜色是白色。安卓全面屏手势的底部安全区是白色。
为了让APP的设计顶部和底部都保持跟设计图一致。下面来说明处理方式。

首先需要明确的是，系统状态栏的背景颜色可以随意更改，文字的颜色只有两种：`深色`和`浅色`，底部的安全区上层为操作系统。
默认代码中系统状态栏和底部安全区中间的部分，就是我们APP的UI展示位置。想要做到将状态栏和底部安全区完全沉浸，需要借助谷歌的另一个库
[accompanist](https://google.github.io/accompanist/)，是为jetpack compose开发人员提供的常用的库。

## 实现
1. 首先引入accompanist库，具体版本可查看官网文档`说明。
   ```gradle
   // 状态栏
   implementation "com.google.accompanist:accompanist-systemuicontroller:0.23.1"
   implementation "com.google.accompanist:accompanist-insets-ui:0.23.1"
   implementation "com.google.accompanist:accompanist-insets:0.23.1"
   ```
2. 设置全屏沉浸（顶部系统状态栏沉浸，底部安全区沉浸）
   ```kotlin
   WindowCompat.setDecorFitsSystemWindows(window, false)
   ```
3. 设置了全屏沉浸后，使用accompanist提供的api，获取系统状态栏高度，底部安全区高度。并设置给顶部和底部padding。
   ```kotlin
   val systemUiController = rememberSystemUiController()   // 获取状态栏API
   
   // 设置底部安全区
   systemUiController.setSystemBarsColor(color = Color.Transparent)
   
   // 设置顶部状态栏
   systemUiController.setStatusBarColor(
       color = Color.Transparent,            // 设置系统状态栏颜色
       darkIcons = !isSystemInDarkTheme(),   // 设置系统状态栏文字是深色还是浅色
   )
   ProvideWindowInsets {
       val insets = LocalWindowInsets.current
       // 获取状态栏高度
       val statusBarHeight = with(LocalDensity.current) { insets.statusBars.top.toDp() }
       // 获取底部安全区高度
       val navigationBarsHeight = with(LocalDensity.current) { insets.navigationBars.bottom.toDp() }

       Scaffold(
           topBar = {
               TopAppBar(
                   // 设置TopAppBar上内边距。
                   contentPadding = PaddingValues(top = statusBarHeight),
               ) {
                    Text(
                        text = "首页",
                        modifier = Modifier
                            .fillMaxWidth()
                            .wrapContentSize(Alignment.Center),
                        fontSize = 18.sp
                    )
               }
           },
           // TODO...
           bottomBar = {
               BottomAppBar(
                   contentPadding = PaddingValues(bottom = navigationBarsHeight),
               ) {
                    // TODO
               }
           }
      )
   }
   ```
