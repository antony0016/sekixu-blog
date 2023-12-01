+++
title = '建立我的第一個 EC2'
date = 2023-12-01T20:55:22+08:00
draft = false
categories = ['筆記']
tags = ['aws', 'ec2']
+++

## 目的

紀錄建立一個 EC2 實體時的步驟，因為相較其他家 VPS 多了一些不一樣的設定，所以寫一點簡單的筆記。

## 名詞解釋

- t2-micro: EC2 的其中一種類型，t 是類型，2 是第幾代，micro 是 size。
- Security group: 用來設定資料的傳入以及傳出規則，基本上就是防火牆，只是可以套用多條到一個群組，在建立 Instance 的時候就可以直接套用多個群組。

## 注意事項

- Image 選擇

    比起其他的 VPS，AWS 多了 Mac 和 Amazon Linux 可以選擇，基於我習慣使用 Docker serve 服務，所以選擇 Amazon Linux 或是其他 Linux 都可以。

- Instance Type

    可以直接選擇 t2-micro，因為免費XD

- Key pair (login)

    用於 SSH login 時的驗證，在這裏可以比較簡單的建立和管理。

- Network setting

    可以選擇要對這個 Instance 建立哪些規則，也可以記得建立 Security group 直接將先前建立過的規則直接套用上來。

- Advance detail - User data

    用來初始化機器用的，輸入指令就可以預先在第一次建立實體時執行指令，像是要不要先安裝好 Web server (like Apache / Nginx)，這個項目在之後的 ECS 會需要用到。
