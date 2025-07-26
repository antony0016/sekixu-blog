+++
title = "K8S - Pods"
date = 2024-11-20T12:01:22+08:00
draft = false
categories = ['Server']
tags = ['Kubernetes', 'k8s']
aliases = ['/posts/k8s/2024_11_20_k8s_pods/', '/posts/k8s/2024_11_20_k8s_pods/index.html']
+++

## 前言

這篇的主題是建立 pod 並且與你建立的 pod 互動，如果需要跳到別的部分的 k8s 筆記，請透過下方連結跳轉~

[K8S - 基礎學習筆記目錄](/posts/k8s/2024_11_20_k8s_basic_note_index/)

## 建立 Pods

```yaml
apiVersion: v1
# 定義這個物件的類型為 Pod
kind: Pod
# 設定這個 Pod 的 metadata(唯一識別資訊)
metadata:
  name: py-backend-test
# 設定這個 Pod 的 spec(規格)
spec:
  # 定義這個 Pod 要運行的容器
  containers:
  # 定義容器的內容，可以把他想像成 docker-compose 的設定
  - name: py-backend-test
    image: sekixu/py-backend-test:latest
    # 定義容器的資源限制，避免這個容器使用過多資源影響到其他容器
    resources:
      # 定義容器的最大資源限制
      limits:
        memory: "512Mi"
        cpu: "500m"
      # 定義容器的最小資源限制
      requests:
        memory: "256Mi"
        cpu: "250m"
    ports:
    # 定義容器要開啟的 Port
    - containerPort: 8000
```

## 指令

```bash
# 套用寫好的 yaml 檔建立 pods
kubectl apply -f pod.backend.yaml

# 取得 pods 資訊
kubectl get pods
# example:
# NAME              READY   STATUS    RESTARTS   AGE
# py-backend-test   1/1     Running   0          119m

# 設定 pods 的 port mapping，讓你可以在本機測試
kubectl port-forward py-backend-test 8000:8000

# ---
# 其他指令: logs, exec, delete
# 查看 pod logs
kubectl logs -f py-backend-test
# 進入 pod
kubectl exec -it py-backend-test bash
# 直接指定 pod name 移除 pods
kubectl delete pod py-backend-test
# 指定檔案移除透過該檔案建立的 pods
kubectl delete -f pod.backend.yaml
```

## 結語

透過這篇可以看到大部分的操作都跟 docker 差不多，但是只會 pods 的話還沒有碰到 k8s 最精華的資源調度，下一篇就來講負責資源調度的 development !