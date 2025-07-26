+++
title = '在 Blog 中加入搜尋的小工具'
date = 2023-11-30T18:30:00+08:00
draft = false
categories = ['Blog']
tags = ['hugo', 'theme', 'stack', 'widget']
aliases = ['/posts/hugo/2023_11_30_add_widget_in_hugo_stack/', '/posts/hugo/2023_11_30_add_widget_in_hugo_stack.html']
+++


## 前言

在我使用的 Stack Theme 中可以使用各種小工具，像是文章類別或是標籤雲等等，但是加入文章搜尋器需要多幾個步驟，稍微研究了一下，簡單記錄一下如何加入。

## 步驟

1. 建立 `content/page/search` 的資料夾。
2. 在 `content/page/search` 中建立 index.md 輸入以下內容

```yaml
---
title: "Search"
layout: "search"
outputs:
    - html
    - json
# 如果想在 sidebar 下面加上 search page link
# 把下面這段取消註解
# menu:
#     main:
#         weight: -60
#         params: 
#             icon: search
---
```

1. 修改 `themes/hugo-theme-stack/config.yaml` 如下

```yaml
params:
  widgets:
      homepage: 
          - type: search
```

1. 最後重新開啟伺服器就可以看到小工具加成功了🎉
