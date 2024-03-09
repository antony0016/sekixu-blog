+++
title = 'hugo 主題設定檔'
date = 2023-11-30T18:00:00+08:00
draft = false
categories = ['Blog']
tags = ['hugo']
+++

## 前言

原本以為設定檔只能在主題的 `config.yaml` 中設定，後來發現可以統一併到 `hugo.toml` 裡面，但是 stack 這個主題好像沒有提供 toml 語法的設定檔，所以我修改好了一份 toml 格式的設定檔。

## 範本

```toml
baseURL = "https://sample.base-url.com/"
languageCode = "zh-tw"
title = "sample title"
theme = "hugo-theme-stack"

[params]
mainSections = ["posts"]
featuredImageField = "image"
rssFullContent = true

[params.footer]
since = 2023

[params.dateFormat]
published = "2006-01-02"
lastUpdated = "2006-01-02"

[params.sidebar]
compact = false
emoji = ""
subtitle = ""

[params.sidebar.avatar]
enabled = true
local = true
src = "images/avatar.png"

[params.article]
readingTime = true

[params.comments]
enabled = true
provider = "gitalk"

[params.comments.gitalk]
owner = ""
admin = ""
repo = ""
clientID = ""
clientSecret = ""

[[params.widgets.homepage]]
type = "search"

[[params.widgets.homepage]]
type = "categories"
[params.widgets.homepage.params]
limit = 10

[[params.widgets.homepage]]
type = "tag-cloud"
[params.widgets.homepage.params]
limit = 10

[colorScheme]
toggle = true
default = "light"

[[menu.social]]
identifier = "github"
name = "GitHub"
url = "https://github.com/xxx/"
[menu.social.params]
icon = "brand-github"
```
