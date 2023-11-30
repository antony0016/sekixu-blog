+++
title = '用 Hugo 建立個人技術部落格'
date = 2023-11-30T17:00:00+08:00
draft = false
categories = ['筆記']
tags = ['hugo']
+++


## 目的

只透過 notion 寫筆記好像沒辦法讓自己的筆記有人看到，所以想試試看用 `hugo` 會不會比較有動力

## 選擇

我參考的是這位[大大提供的比較](https://raychiutw.github.io/2019/Static-Site-Generator-Comparison/)，由於只需要最簡單的 Markdown，也不需要其他多餘的功能，所以就選擇目前最多星星的 `Hugo` 了

## 步驟

1. 先到 [Hugo 的官網](https://gohugo.io/getting-started/quick-start/)安裝好 `hugo` 和 `git`

痾.. 就這樣 😀

## 指令

- 建立新網站

```bash
hugo new site quickstart
cd quickstart
git init
```

- run local server

本來想說想打中文，但是”運行本地伺服器”實在太怪了 🤣

```bash
# run server without draft
hugo server
# run server with draft
hugo server -D
```

- 新增主題

```bash
# sample command
git submodule add <github-url> themes/<theme-name>
echo "theme = '<theme-name>'" >> hugo.toml

# sample
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
```

- 新增文章

```bash
hugo new content posts/<file-name>.md
```

- 文章範本

```bash
+++
title = "My First Post"
date = 2022-11-20T09:03:20-08:00
draft = true
+++
## Introduction

This is **bold** text, and this is *emphasized* text.

Visit the [Hugo](https://gohugo.io) website!
```

- publish

```bash
hudo
```

## 主題內容設定

請參考各個主題的官方文件，我使用的是 stack 這個主題 [link](https://stack.jimmycai.com/config/)
