+++
title = '加入留言系統到 blog 中'
date = 2023-11-30T19:41:43+08:00
draft = false
categories = ['Blog']
tags = ['hugo', 'theme', 'stack', 'utterances']
+++

## 前言

仔細看了一下 stack 這個主題還能加入什麼小工具，發現它支援很多不同的留言系統，在簡單比較之後決定使用 Utterances ，最簡單也可以整合到 GitHub 上。

## 步驟

1. 建立一個 Public GitHub Repo
2. 在 `config.yaml` or `hugo.toml` 中修改以下內容

```yaml
params:
comments:
    enabled: true
    provider: utterances
    utterances:
        repo: <github-username>/<github-repo-name>
        issueTerm: og:title
        theme: github-dark
```

```toml

[params.comments]
enabled = true
provider = "utterances"

[params.comments.utterances]
repo = "<github-username>/<github-repo-name>"
issueTerm = "og:title"
theme = "github-dark"
```

## 參考資源

[Utteranc 官方網站](https://utteranc.es/)
