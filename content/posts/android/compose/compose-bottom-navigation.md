---
categories: ["Android", "Jetpack", "Compose"]
date: "2022-08-08"
description: "Compose整合BottomBar和NavigationBar"
title: "Compose整合BottomBar和NavigationBar"
type: "post"
---

## 概述
大致看了下现有的一些Compose整合底部导航的文章，大部分都是使用的**BottomBar** + **State**实现。此操作虽然能够实现底部导航切换，但是跟市面上的APP还是有部分差距。

- 没有跟导航图结合，自行实现的底部导航，需要写大量的业务逻辑进行判断。 
- 已打开过的页面，再次打开会重新渲染，降低性能。 
- 仔细观察市面上APP，如B站，京东等。当切换至其他页面时，点击返回按钮，首先会退出到首页，再次返回才是退出APP。而自行实现的，需要手动处理此类需求。

其实**Material**已经完美的整合了此功能。下面来说明如何使用Compose来整合底部导航。

## 1. 准备页面
底部导航有几个切换条，就需要准备几个页面。为了演示方便，我准备了三个页面进行演示：`HomeScreen（首页）`，`FindScreen（发现）`，`MineScreen（我的）`。
```kotlin
// 首页
@Composable
fun HomeScreen() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(color = Color.Blue),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "首页",
            fontSize = 18.sp,
            color = Color.White
        )
    }
}

// 发现页
@Composable
fun FindScreen() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(color = Color.Green),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "发现",
            fontSize = 18.sp,
            color = Color.White
        )
    }
}

// 我的
@Composable
fun MineScreen() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(color = Color.Yellow),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "我的",
            fontSize = 18.sp,
            color = Color.White
        )
    }
}
```

## 2. 定义底部导航
此处使用kotlin的关键字`sealed`来修饰，表示此类是一个密封类。

```kotlin
sealed class BottomBarScreen(
    val route: String,
    val title: String,
    val icon: ImageVector
) {
    object Home: BottomBarScreen(
        route = "home",
        title = "首页",
        icon = Icons.Default.Home
    )

    object Find: BottomBarScreen(
        route = "find",
        title = "发现",
        icon = Icons.Default.Search
    )

    object Mine: BottomBarScreen(
        route = "mine",
        title = "我的",
        icon = Icons.Default.Person
    )
}
```

## 3. 创建导航图
创建导航图，导航图绑定页面和路由，并设置导航图启动后，第一个导航的页面。

```kotlin
@Composable
fun BottomNavGraph(navController: NavHostController) {
    NavHost(
        navController = navController,
        startDestination = BottomBarScreen.Home.route
    ) {
        composable(route = BottomBarScreen.Home.route) {
            HomeScreen()
        }

        composable(route = BottomBarScreen.Find.route) {
            FindScreen()
        }

        composable(route = BottomBarScreen.Mine.route) {
            MineScreen()
        }
    }
}
```

## 4. 创建主页面
主页面中我们使用`Scaffold`创建基础布局。并创建我们自定义的底部`BottomBar`，其中自定义的`BottomBar`。
**注意**：代码示例是使用Material3进行编写，如果使用的是Material2，需要将`NavigationBar`更改为`BottomBar`，将`NavigationBarItem`更改为`BottomBarItem`。
```kotlin
// 主页面
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    Scaffold(
        bottomBar = {
            BottomBar(navController = navController)
        }
    ) {
        BottomNavGraph(navController = navController)
    }
}
```

## 5. 底部导航
因为代码是使用`Material3`编写，此处的`BottomBar`是为了迎合2版本来写的示例。切不可与官方的`BottomBar`混淆。

```kotlin
// 自定义BottomBar
@Composable
fun BottomBar(navController: NavHostController) {
    val screens = listOf(
        BottomBarScreen.Home,
        BottomBarScreen.Find,
        BottomBarScreen.Mine
    )

    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentDestination = navBackStackEntry?.destination

    NavigationBar {
        screens.forEach { screen ->
            // 自定义单个Item
            AddItem(
                screen = screen,
                currentDestination = currentDestination,
                navController = navController
            )
        }
    }
}

@Composable
fun RowScope.AddItem(
    screen: BottomBarScreen,
    currentDestination: NavDestination?,
    navController: NavHostController
) {
    NavigationBarItem(
        label = {
            Text(text = screen.title)
        },
        icon = {
            Icon(
                imageVector = screen.icon, contentDescription = screen.title
            )
        },
        selected = currentDestination?.hierarchy?.any {
            it.route == screen.route
        } == true,
        onClick = {
            navController.navigate(screen.route) {
                // 重点：以下为官方提供的处理方案。
                // 弹出到起始的home页，避免建立一堆的目的地。
                popUpTo(navController.graph.findStartDestination().id) {
                    saveState = true
                }
                // 避免同一个目的地建立多个副本。
                launchSingleTop = true

                // 重新打开已经打开的目的地时，恢复其状态
                restoreState = true
            }
        }
    )
}
```

## 结束
以上就是Compose整合Navigation的方法。此方法是官方所推荐的。
