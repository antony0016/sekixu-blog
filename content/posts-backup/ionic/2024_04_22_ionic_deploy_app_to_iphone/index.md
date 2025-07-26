+++
title = 'Ionic 部署 - ios 版本'
date = 2024-04-22T23:39:00+08:00
draft = false
categories = ['Frontend']
tags = ['ionic', "capacitor"]
+++

## 前言

Ionic 使用的是 Capacitor，是一套可以用來將打包好的網頁包成 App 的工具，這篇將示範在 m 晶片的 MacBook 上測試編譯 ios 版本的 App，另外這篇會提到一下我編譯過程出現的問題，如果照著這篇有問題可以參考看看。

## 文件參考

這篇會用到以下兩篇官方文件

[環境準備](https://capacitorjs.com/docs/getting-started/environment-setup)

[建置流程](https://ionicframework.com/docs/cli/commands/capacitor-run?_gl=1*1qiy9mh*_ga*MTQyNDI2MzcyNS4xNzEzNjQzMTk4*_ga_REH9TJF6KF*MTcxMzcxMjc2Ni4yLjEuMTcxMzcxNjg0NC4wLjAuMA..)

## 步驟

1. 編譯環境準備
2. 為 Ionic 專案安裝 capacitor
3. 在專案中加入所需系統 ios / android
4. 打開 xcode / 使用 capacitor 指令測試

## Step 1 - 編譯環境準備

由於先前就有大部分要求的編譯環境了，所以這裡我會簡單帶過。

- Node.js : [官網](https://nodejs.org/en/download)

到官網下載 mac 版 Node.js，像一般應用程式安裝方式。

- homebrew: [官網](https://brew.sh/)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- CocoaPods: [官網](https://cocoapods.org/)

也可以用 gem(ruby package manager)，但這邊推薦用 homebrew 安裝，因為用 gem 安裝在apple silicon mac 上安裝會有架構問題，需要另外疑難排解。

```bash
brew install cocoapods
```

- Xcode & Xcode Command Line Tools: [App store](https://apps.apple.com/us/app/xcode/id497799835?mt=12)

到 App store 下載 Xcode，安裝完之後因為預設不包含 Xcode Command Line Tools，需要輸入下列指令安裝。

```bash
# 安裝 Xcode Command Line Tools
xcode-select --install

# 驗證安裝
xcode-select -p
# /Applications/Xcode.app/Contents/Developer
```

## Step 2 - 為 Ionic 專案安裝 capacitor

如果是新版本的 ionic，預設安裝就已經會有 capacitor，但是如果是舊專案的話，請使用以下 ionic CLI 指令啟用 Capacitor。

```bash
ionic integrations enable capacitor

# This command will install following dependencies
# @capacitor/app
# @capacitor/haptics
# @capacitor/keyboard
# @capacitor/status-bar
```

## Step 3 - 在專案中加入所需系統 ios / android

透過輸入下面的指令，可以在專案中加入 ios 平台的 Xcode 專案，這會用來在後面啟動專案，把 app 輸出到模擬器或是你自己的 iPhone 使用並測試。

```bash
ionic capacitor add ios
```

## Step 4 - 打開 xcode / 使用 capacitor 指令測試

由於我遇到了一些環境問題，所以我需要經常用到 open 的指令打開 Xcode 專案進行偵錯及修正，所以會需要下面這個指令。

```bash
ionic capacitor open ios
```

在最完美狀況下，應該要使用以下 run 指令就可以選擇模擬器 / iPhone 進行 app 部署及測試，但是正常來說會遇到一些環境及設定問題，請參照注意事項進行修正後，回來重跑一次這個指令，全部處理完畢以後就可以再重新執行指令測試。

```bash
ionic capacitor run ios
```

## 注意事項

### 1. 注意 xcode 預設位置

在安裝 Xcode 時，我們會輸入 `xcode-select -p` 驗證安裝位置，如果像我一樣因為先前專案有切換到不同地方，就可以按照下一個指令(-s)切換到正確的目錄。

```bash
xcode-select -p
# 正確結果: /Applications/Xcode.app/Contents/Developer
# 錯誤結果: /Library/Developer/CommandLineTools
```

How to fix:

```bash
xcode-select -s /Applications/Xcode.app/Contents/Developer
# 切換到正確目錄
# -s switch
```

### 2. 使用模擬器測試 app

在輸入 `ionic capacitor run ios` 應該要可以看到你有哪些裝置可以使用，在裝置列表中如了你連結到電腦的 iPhone 以外，也會有 iPhone 15, iPhone 15 pro, iPhone SE 等等模擬器可以使用。

如果沒有出現請先確認第一點的 Xcode 位置是否正確，再嘗試以下解法。

How to fix:

Xcode > Platforms > 勾選 ios 17.x 進行安裝

### 3. 在 Xcode 中登入 apple 帳號

在 Xcode 中，你會需要登入你的 apple 帳號，作為你的開發者帳號將程式部署到 iPhone 上做測試。

How to fix:

Xcode > Accounts > 登入 Apple ID > 選擇 Personal Team。

### 4. 在 iPhone 上開啟開發者模式

要在 iPhone 上測試 App，會需要先開啟開發者模式，才不會在選擇裝置的時候，一直出現裝置重連的狀況。

How to fix:

iPhone > 隱私性與安全性 > (拉到最下面) > 開發者模式 > (開啟並重開機)

### 5. 在 iPhone 上信任開發者

如果你選擇 iPhone 測試 App，你會遇到打不開 App 的問題，因為你的 App 沒上 App store，iPhone 不認識裝了這個 App 的開發者，所以需要手動信任這個開發者。

如果我說錯了請糾正我XDD

How to fix:

iPhone > 一般 > (拉到最下面) > VPN 與裝置管理 >

開發者 App > 找到你的開發者帳號 > 信任這個開發者

這樣就可以順利打開 App 了~

## 總結

第一次碰 ionic 這個框架，可以用網頁的方式寫 App，雖然什麼都還沒寫，但是可以順利跑在手機上還是覺得很有成就感，不知道這次的系統會有怎麼樣的需求，如果有遇到值得紀錄的點，我再寫文章記錄下來。
