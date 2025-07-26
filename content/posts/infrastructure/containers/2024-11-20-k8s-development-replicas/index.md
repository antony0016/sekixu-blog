+++
title = "K8S - Development - Replicas"
date = 2024-11-20T14:01:22+08:00
draft = false
categories = ['Server']
tags = ['Kubernetes', 'k8s']
aliases = ['/posts/k8s/2024_11_20_k8s_development_replicas/', '/posts/k8s/2024_11_20_k8s_development_replicas/index.html']
+++

## 前言

這篇的主題是建立 development 並且確認 development 在各個狀況會如何調度資源，如果需要跳到別的部分的 k8s 筆記，請透過下方連結跳轉~

[K8S - 基礎學習筆記目錄](/posts/k8s/2024_11_20_k8s_basic_note_index/)

## 建立 Development

```yaml
apiVersion: apps/v1
# 定義這個物件的類型為 Deployment
kind: Deployment
# 設定這個 Deployment 的 metadata(唯一識別資訊)
metadata:
  name: py-backend-test-deployment
# 設定這個 Deployment 的 spec(規格)
spec:
  # 定義這個 Deployment 要運行的 Pod 的數量
  replicas: 2
  # 定義這個 Deployment 如何去選擇要管理的 Pod
  # 對應到 Pod 的 metadata.labels
  selector:
    matchLabels:
      app: py-backend-test
  # 定義這個 Deployment 要管理的 Pod 的模板
  template:
    # 定義這個 Pod 的 metadata(唯一識別資訊)
    # 這邊的 metadata.labels 會對應到 Deployment 的 selector.matchLabels
    metadata:
      labels:
        app: py-backend-test
    spec:
      containers:
      - name: py-backend-test
        image: sekixu/py-backend-test:latest
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
        ports:
        - containerPort: 8000
```

## 指令

```bash
# 套用寫好的 yaml 檔建立 development
kubectl apply -f development.backend.yaml

# 取得 development 資訊
kubectl get pods
# example:
# NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
# py-backend-test-deployment   2/2     2            2           12s

# 取得由 development 建立的 pods 資訊
kubectl get pods
# example:
# NAME                                          READY   STATUS    RESTARTS   AGE
# py-backend-test-deployment-64c6765877-6mm7b   1/1     Running   0          21s
# py-backend-test-deployment-64c6765877-mx5zx   1/1     Running   0          21s
```

## 資源調度

嘗試刪除掉其中一個 pods 並觀察 pod 數量，可以發現下面範例 `6mm7b` 結尾的 container 被刪除後，development 自動幫我們建立了 `28xtj` 結尾的 pod。

```bash
# delete pod
kubectl delete pod py-backend-test-deployment-64c6765877-6mm7b

# check pod status
kubectl get pods
# NAME                                          READY   STATUS    RESTARTS   AGE
# py-backend-test-deployment-64c6765877-28xtj   1/1     Running   0          34s
# py-backend-test-deployment-64c6765877-mx5zx   1/1     Running   0          7m17s
```

## 結語

透過這篇可以看到 development 如何建立多個 pods 並且自動管理，其實這還只是 development 最基礎的一個功能，這個章節先學到這，後續再把 development 更強的功能拉進來一起講。