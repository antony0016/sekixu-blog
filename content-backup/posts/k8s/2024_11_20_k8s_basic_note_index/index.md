+++
title = "K8S - 基礎學習筆記目錄"
date = 2024-11-20T10:01:22+08:00
draft = false
categories = ['Server']
tags = ['Kubernetes', 'k8s']
+++

## 前言

這個文章將會一步一步帶你把一個簡單的前後端分離的服務部署上本地的 K8S(with Docker CE)，參考教學文章為 [guangzhengli/k8s-tutotials](https://github.com/guangzhengli/k8s-tutorials/)，感謝這個作者開了這個相當好懂的 K8S 教學 Repo。

這個文章內所有的步驟，都會透過我自己的 Repo [matsuno.seki/eks-cicd-template](https://gitlab.com/matsuno.seki/eks-cicd-template) 來實作一遍，大家也可以自己開一個 Repo，針對想練習的部分實作看看!

## 章節

在這個文章我會跳過服務建立的部分，大家可以動動腦袋把自己的服務編成 container 推上 docker hub 後一起實作看看，另外在每個步驟後面的括號都是我自己為了更好理解打上去的註解，如果哪邊有錯或是誤解的話可以跟我說~

- 建立 pods(set of containers)
    - [k8s - Pods](/posts/k8s/2024_11_20_k8s_pods/)
- 建立 deployment(auto pods management)
    - [k8s - development - replicas](/posts/k8s/2024_11_20_k8s_development_replicas/)
- 建立 service(pods proxy)

- 建立 ingress controller(service gateway)


## 參考文件
[guangzhengli/k8s-tutotials](https://github.com/guangzhengli/k8s-tutorials/)