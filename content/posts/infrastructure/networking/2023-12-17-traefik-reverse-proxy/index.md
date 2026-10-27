+++
title = '使用 Traefik 作為反向代理伺服器'
date = 2023-12-17T03:51:49+08:00
draft = false
categories = ['Infrastructure']
tags = ['docker', 'traefik']
aliases = ['/posts/traefik/2023_12_17_traefik_reverse_proxy/', '/posts/traefik/2023_12_17_traefik_reverse_proxy/index.html']
+++

## 前言

使用 Traefik 可以很簡單的建立反向代理伺服器，並且自動獲取 SSL 證書，所以嘗試記錄一下。

> 本文寫於 2023-12，用的是 Traefik v2.10。Traefik 現在已經是 v3，升級前請先看官方的 v2 到 v3 遷移文件。

## 預備步驟

安裝 docker & docker-compose 在伺服器上。

## 步驟

- 建立 Docker 網路
- 準備好 Traefik 設定檔
- 建立 Docker compose file
- 設定反向代理

## 建立 Docker 網路

透過以下指令建立 docker network，在後面的 docker compose file 會用到。

```docker
docker network create <network-name>
# docker network create traefik_network
```

## 準備 Traefik 設定檔

先寫好 Traefik 設定檔，檔名為 traefik.yaml，要放在 /etc/traefik 中，這裡的設定檔沒記錯應該有參考到網路上的某篇文章，知道是哪篇的或是作者再麻煩聯繫我，我會標注的 QQ。

```yaml
api:
  # 關閉 Traefik 8080 port
  insecure: false
  # 啟用 Traefik 的 dashboard
  dashboard: true

# 設定 Traefik 的 providers
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false

# 啟用 ACME (Let's Encrypt) 功能
certificatesResolvers:
  letsencrypt-production:
    acme:
      email:  # 設定你的 email
      storage: /etc/traefik/acme/acme.json # 設定 ACME 資料儲存路徑
      # caServer: https://acme-v02.api.letsencrypt.org/directory # production ca server
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory # test ca server
      
      tlsChallenge: {}

      # httpChallenge:
      #   entryPoint: https # 設定使用 HTTP Challenge 驗證
  
# 設定 Traefik 的 entryPoints
entryPoints:
  https:
    address: ":443"
```

## 建立 Docker compose file

透過以下的 docker compose file 填入對應的欄位建立 traefik 的容器，配合下方的 env 填入機敏資訊。

注意事項

- networks 要設定連接到剛剛建立的 network，為了給其他容器加入用。

```yaml
services:
  traefik:
    image: "traefik:v2.10"
    container_name: "traefik"
    env_file:
      - ".env"
    ports:
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "$TRAEFIK_DATA:/etc/traefik:ro"
      - "$TRAEFIK_DATA/acme:/etc/traefik/acme:rw"
    labels:
      - "traefik.enable=true"
   # https
      # - "traefik.http.routers.traefik_https.rule=Host(`$TRAEFIK_SUBDOMAIN.$BASE_DOMAIN`)"
      # - "traefik.http.routers.traefik_https.entrypoints=https"
      # - "traefik.http.routers.traefik_https.tls.certresolver=letsencrypt-production"
      # - "traefik.http.routers.traefik_https.tls=true"
     # - "traefik.http.routers.traefik_https.service=api@internal"
    networks:
      - traefik_network

networks:
  traefik_network:
    external: true
```

## 設定反向代理

以 Nginx 作為範例，只要在 docker compose 中的 labels，加上這些標籤就可以設定反向代理。

注意事項

- `traefik.http.routers.<router-name>` 請將 router-name 換成自定義的 router name，本例為 test。
- `traefik.http.routers.<router-name>.tls.certresolver` 要填入設定檔中的 resolver 名稱。
- networks 要設定連接到剛剛建立的 network，才可以被 traefik 偵測到。

```yaml
nginx-test:
    image: "nginx"
    container_name: "test"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.test.rule=Host(`$TEST_SUBDOMAIN.$BASE_DOMAIN`)"
      - "traefik.http.routers.test.entrypoints=https"
      - "traefik.http.routers.test.tls.certresolver=letsencrypt-production"
      - "traefik.http.routers.test.tls=true"
    networks:
      - traefik_network

networks:
  traefik_network:
    external: true
```

## 結語

Traefik 有更多細節與功能可以設定，但是第一次接觸實在需要比較多的範例與試錯，這篇文是整整搞了三天的精華，有問題可以一起討論(因為我也不一定會😀)。
