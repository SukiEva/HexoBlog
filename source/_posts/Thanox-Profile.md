---
title: Thanox 情景模式
toc: true
date: 2021-11-17 10:04
tags:
  - Thanox
categories: Mixed
updated: 2022-06-25 16:26
references:
  - title: Thanox 情景模式官方文档
    url: https://tornaco.github.io/Thanox/6-Profile.html
---

情景模式简单理解：

> 什么时候，干什么。

- 什么时候就是触发条件（condition）
- 干什么就是动作（actions）

本文分享自用的情景模式，不定期更新~

<!-- more -->

## GPS

### 自动开关

```json
[
    {
        "name": "Auto GPS",
        "description": "全局变量gps",
        "priority": 1,
        "condition": "(frontPkgChanged && globalVarOf$gps.contains(to) && !hw.isLocationEnabled()) || (pkgKilled && globalVarOf$gps.contains(pkgName) && hw.isLocationEnabled())",
        "actions": [
            "if (!hw.isLocationEnabled()) hw.enableLocation(); else hw.disableLocation();"
        ]
    }
]
```

### 微信小程序 & 内置网页

```json
[
    {
        "name": "Wechat GPS",
        "description": "小程序和内置网页",
        "priority": 1,
        "condition": "(activityResumed && !hw.isLocationEnabled() && (componentNameAsString.contains('.appbrand.ui.') || componentNameAsString.contains('.plugin.webview.'))) || (frontPkgChanged && hw.isLocationEnabled() && from == 'com.tencent.mm')",
        "actions": [
            "if (!hw.isLocationEnabled()) hw.enableLocation(); else hw.disableLocation();"
        ]
    }
]
```

## NFC

### 自动开关

```json
[
    {
        "name": "Auto NFC",
        "description": "全局变量nfc",
        "priority": 1,
        "condition": "(frontPkgChanged && globalVarOf$nfc.contains(to) && !hw.isNfcEnabled()) || (pkgKilled && globalVarOf$nfc.contains(pkgName) && hw.isNfcEnabled())",
        "actions": [
            "if (!hw.isNfcEnabled()) hw.enableNfc(); else hw.disableNfc();"
        ]
    }
]
```

## 冻结相关

### 蓝牙开关冻结

```yaml
name: "BT冻结"
description: "蓝牙关闭冻结指定应用，全局变量 btkill"
priority: 2
condition: "btStateChanged && btStateOff"
actions:
    - "for (String s : globalVarOf$btkill) { if (pkg.isApplicationEnabled(s)) pkg.disableApplication(s) }"
    - 'ui.showShortToast("🎉BT冻结")'
```

```yaml
name: "BT解冻"
description: "蓝牙打开解冻指定应用，全局变量 btkill"
priority: 2
condition: "btStateChanged && btStateOn"
actions:
    - "for (String s : globalVarOf$btkill) { if (!pkg.isApplicationEnabled(s)) pkg.enableApplication(s) }"
    - 'ui.showShortToast("🎉BT解冻")'
```

### 游戏开关冻结

> 自行修改以下 com.oneplus.gamespace 包名

```yaml
name: "Game冻结"
description: "关闭游戏冻结游戏空间，全局变量 game"
# 一加专属
priority: 2
condition: 'taskRemoved && globalVarOf$game.contains(pkgName) && pkg.isApplicationEnabled("com.oneplus.gamespace")'
actions:
    - 'pkg.disableApplication("com.oneplus.gamespace")'
    - 'ui.showShortToast("🎉冻结游戏空间")'
```

```yaml
name: "Game解冻"
description: "打开游戏解冻游戏空间，全局变量 game"
# 一加专属
priority: 2
condition: 'frontPkgChanged && globalVarOf$game.contains(to) && !pkg.isApplicationEnabled("com.oneplus.gamespace")'
actions:
    - 'pkg.enableApplication("com.oneplus.gamespace")'
    - 'ui.showShortToast("🎉解冻游戏空间")'
```

## 应用相关

### APP 保活

> 推荐用乖巧模式的规则来 KEEP 想保持的服务
>
> 以下貌似没用，可以开启电池不优化试试

```yaml
name: "APP保活"
description: "应用停止运行时重启应用进程，全局变量 apps"
#  APP 后台不优化
priority: 2
condition: "pkgKilled && globalVarOf$apps.contains(pkgName)"
actions:
    - "activity.launchProcessForPackage(pkgName)"
    - 'ui.showShortToast("🎉保活app")'
```

### APP 休眠

```yaml
name: "APP休眠"
description: "后台应用休眠，全局变量 idle"
priority: 2
condition: "frontPkgChanged && globalVarOf$idle.contains(from)"
actions:
    - "activity.setInactive(from)"
```

## Data 相关

### Data 自动开启

```yaml
name: "Data自动开启"
description: "打开应用打开移动数据，全局变量 data"
priority: 2
condition: "frontPkgChanged && globalVarOf$data.contains(to) && !hw.isWifiEnabled() && !data.isDataEnabled()"
actions:
    - "data.setDataEnabled(true)"
    - 'ui.showShortToast("🎉打开移动数据")'
```

## 亮度相关

### 自动亮度关闭

```yaml
name: "自动亮度关闭"
description: "打开应用关闭自动亮度，全局变量 bright"
priority: 2
condition: "frontPkgChanged && globalVarOf$bright.contains(to) && power.isAutoBrightnessEnabled()"
actions:
    - "power.setAutoBrightnessEnabled(false)"
    - 'ui.showShortToast("🎉关闭自动亮度")'
```

### 自动亮度开启

```yaml
name: "自动亮度开启"
description: "停止应用打开自动亮度，全局变量 bright"
priority: 2
condition: "pkgKilled && globalVarOf$bright.contains(pkgName) && !power.isAutoBrightnessEnabled()"
actions:
    - "power.setAutoBrightnessEnabled(true)"
    - "power.setBrightness(power.getBrightness())"
    - 'ui.showShortToast("🎉开启自动亮度")'

```

## 状态栏图标相关

> 隐藏状态栏图标的 shell 命令：
>
> settings put secure icon_blacklist 「args」
>
> 自行修改 「args」 参数

### 状态栏图标隐藏

> 仅供参考，需要 su 插件

```yaml
name: "状态栏图标隐藏"
description: "打开应用隐藏状态栏图标,全局变量 bar"
priority: 2
condition: "frontPkgChanged && globalVarOf$bar.contains(to)"
actions:
    - 'su.exe("settings put secure icon_blacklist volte,vowifi,rotate,zen,clock,battery,vpn")'
    - 'ui.showShortToast("🎉隐藏状态栏图标")'
```

### 状态栏图标显示

```yaml
name: "状态栏图标显示"
description: "停止应用显示状态栏图标,全局变量 bar"
priority: 2
condition: "pkgKilled && globalVarOf$bar.contains(pkgName)"
actions:
    - 'su.exe("settings put secure icon_blacklist volte,vowifi,rotate,zen,battery,vpn")'
    - 'ui.showShortToast("🎉显示状态栏图标")'
```
