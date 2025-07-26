+++
title = '第一次編譯開源專案 gravitino'
date = 2025-01-08T01:29:00+08:00
draft = false
categories = ['OSS']
tags = ['gravitino', 'oss']
aliases = ['/posts/oss/2025_01_08_gravitino_compile_error/', '/posts/oss/2025_01_08_gravitino_compile_error/index.html']
+++
## 目的

想要嘗試進入開源的世界的我選擇了 Gravitino，我已經簡單碰過 [gravitino-playground](https://github.com/apache/gravitino-playground) 了，有最基本的瞭解了，於是嘗試編譯…但是失敗了!!

讓我們來看看我遇到了什麼問題，以及我是怎麼解決的。

## 步驟

1. 安裝 Java Runtime
2. 編譯 Gravitino
3. 除錯

## 安裝 Java Runtime

首先我是使用 SDKMan 來安裝 Java 的，透過以下指令就可以正常安裝並且管理 Java Runtime。

```bash
# 安裝 sdkman
curl -s "https://get.sdkman.io" | bash

# 全域安裝 openjdk 17.0.13-ms
sdk install java 17.0.13-ms
```

## 編譯 Gravitino

透過以下指令就可以執行編譯 Gravitino 的腳本，前面跑了 4~5 分鐘左右都沒有什麼問題。

```bash
# command to build gravitino
./gradlew clean build -x test
```

直到執行到這個 Task 的時候，我遇到了以下問題。

```bash
> Task :clients:client-python:miniforgeSetup FAILED
```

## 除錯

以下是我嘗試過的解決辦法，但是都失敗了：

1. 換 python 版本 3.8 / 3.11
2. 提前設定好這腳本內用到的環境變數
3. 重新 clone 專案

再來我重新整理了一下我的環境設定和執行結果大概有幾個重點：

1. 我直接執行這個任務用到的腳本可以正常運行
2. 透過 Gradle 執行編譯腳本卻會壞掉

透過以上幾點，我開始懷疑我的 JDK 版本，我透過以下指令將本機 JDK 版本換成了早一個小版本的`17.0.12-tem` 然後就可以正常編譯了。

## 結語

在遇到編譯問題的時候，最好找社群與你有相同 / 類似設備的好夥伴詢問可以順利編譯的環境版本…

如果沒有好夥伴的話，下次記得不要使用最新編譯版本的環境來嘗試編譯，往前一兩個版本可以多少避免這種新版本不穩定的問題。