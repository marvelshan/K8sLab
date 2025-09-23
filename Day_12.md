## K8S Lab Day_12

# Day10: Istio 流量管理的戰術總覽與自動化 SSH 優化

## 前言

昨天做了 istio 比較基本可以做到的功能認證和授權，接著就是比較應用層面的，今天會講到比較應用層面的場景，在微服務架構下，traffic management 是一件很重要的事，今天就稍微來介紹一下吧

另外，我自己在操作的時候因為都要一直連線 ssh，然後每次都要先進到放 `flake.nix` 的資料夾底下，再執行 `nix develop`，所以我就想說優化他，在 `~/.bashrc` 檔案內加上我需要執行的指令

```bash
vi ~/.bashrc
```

```bash
if [ -z "$IN_NIX_DEVELOPMENT" ]; then
  # Set a flag to prevent re-running in the subshell
  echo $IN_NIX_DEVELOPMENT
  export IN_NIX_DEVELOPMENT=true
  if [ -z "$SSH_ORIGINAL_COMMAND" ]; then
    cd ~/kubespray-nix
    /home/ubuntu/.nix-profile/bin/nix develop
  fi
fi
```

將這些放入整個檔案的最後面，這邊的邏輯給大家自己細細品味，其中的 `/home/ubuntu/.nix-profile/bin/nix develop` 這邊不是使用 `nix develop` 是因為 Shell 腳本的執行環境中，nix 這個指令可能不在 $PATH 環境變數裡，當腳本在 ~/.bashrc 裡自動執行時，$PATH 可能還沒有被完整設定，導致 Shell 找不到 nix 這個指令，從而報錯 `Command 'nix' not found`

## Traffic Management

很好，你這樣安排就像是在做 **Istio Traffic Management 的實驗手冊**。
你提到的這幾個功能，基本上就是 **微服務在真實世界運作時的流量控制需求**，每一個都對應到一種「實務場景」。

我幫你把每個主題口語化，並且舉例說明「什麼情況下會用到」，讓你在實作前有一個完整的脈絡：

---

## Traffic Management 功能與場景

### 1. Request Routing

是根據請求的條件（HTTP Header、URL Path、使用者 ID）把流量導到不同版本的服務，應用的場景通常在你要做 A/B 測試 或 Canary 部署

```yaml
spec:
  hosts:
    - reviews
  http:
    - match:
        - headers:
            end-user:
              exact: jason
      route:
        - destination:
            host: reviews
            subset: v2
    - route:
        - destination:
            host: reviews
            subset: v1
```

---

### 2. Fault Injection

在某些強況下會有一些 delay 或 error code

```yaml
spec:
  hosts:
    - ratings
  http:
    - fault:
        abort:
          httpStatus: 500
          percentage:
            value: 100
      match:
        - headers:
            end-user:
              exact: jason
      route:
        - destination:
            host: ratings
            subset: v1
    - route:
        - destination:
            host: ratings
            subset: v1
```

---

### 3. Traffic Shifting

把流量比例分配到不同版本，在部署的時候有時會需要做 Canary release 這種漸進式的部署，像是先給 10% v2 確認穩定後再拉到 50% 最後 100%

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 90
        - destination:
            host: reviews
            subset: v2
          weight: 10
```

然後在調整成

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 50
        - destination:
            host: reviews
            subset: v2
          weight: 50
```

最後

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v2
          weight: 100
```

---

### 4. TCP Traffic Shifting

跟 HTTP 的 Traffic Shifting 一樣，但應用在 TCP 協議（例如資料庫 Proxy），應用場景在你要升級一個 Redis/MongoDB 後端，先導 10% 流量到新 DB，觀察行為

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: redis
spec:
  hosts:
    - redis
  tcp:
    - match:
        - port: 6379
      route:
        - destination:
            host: redis
            subset: v1
            port:
              number: 6379
          weight: 80
        - destination:
            host: redis
            subset: v2
            port:
              number: 6379
          weight: 20
```

---

### 5. Request Timeouts

設定一個請求的等待上限，使用場景通常是避免上游被下游卡死，超過 0.5s 就返回錯誤，主要防止「整個過程因為一個卡住的服務拖垮」

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v2
      timeout: 0.5s
```

---

### 6. Circuit Breaking

如果某個服務回應一直失敗，就暫時不送流量給它，主要是保護整個系統，避免「壞掉的服務拖累所有人」，像是 ratings 連線錯誤率 >50%，就先斷掉，過幾秒再恢復

```yaml
kind: DestinationRule
---
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1 # HTTP 1.1 最大待處理請求數，超過會拒絕
        maxRequestsPerConnection: 1 # 每個 TCP 連線允許的最大 HTTP 請求數
      tcp:
        maxConnections: 1 # TCP 最大連線數
    outlierDetection: # 異常檢測 (Outlier Detection) 設定
      baseEjectionTime: 3m # 異常 pod 被踢掉後等待時間 (3 分鐘)，過後自動恢復
      consecutive5xxErrors: 1 # 連續 5xx 錯誤超過 1 次就將該 pod 暫時踢掉
      interval: 1s # 每 1 秒檢查一次 pod 健康狀態
      maxEjectionPercent: 100 # 最多踢掉 100% 的 pod，避免全數流量被切斷可視情況調整
```

---

### 7. Mirroring

複製一份真實流量給另一個版本，但不影響使用者，應用場景主要是想要線上測試新版本，但不想影響用戶

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
- httpbin
  http:
  - route:
- destination:
    host: httpbin
    subset: v1
  weight: 100
mirror:
  host: httpbin
  subset: v2
mirrorPercentage:
  value: 100.0
```

---

### 8. Locality Load Balancing

盡量把流量導到最近的實例，場景多使用在跨多地部署（台灣、東京、美國），要讓台灣用戶優先打到台灣的服務

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: helloworld
spec:
  host: helloworld.sample.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      localityLbSetting:
        enabled: true
        distribute: # 地區加權
          - from: region1/zone1/*
            to:
              "region1/zone1/*": 70
              "region1/zone2/*": 20
              "region3/zone4/*": 10
    outlierDetection:
      consecutive5xxErrors: 100
      interval: 1s
      baseEjectionTime: 1m
```

---

### 9. 還有 [Ingress](https://istio.io/latest/docs/tasks/traffic-management/ingress/) 和 [Egress](https://istio.io/latest/docs/tasks/traffic-management/egress/)

更細緻的管控外部進來的流量怎麼進出 mesh，完全取決於場景的設計，這裏就不多說了～

## 結論

因為真的蠻多種應用場景的，大家可以花些時間去玩玩看文件上面的一些流程，會更加認識 DestinationRule 和 VirtualService 的功能的～

## Reference

https://ithelp.ithome.com.tw/articles/10301327

https://ithelp.ithome.com.tw/articles/10301329

https://medium.com/brobridge/%E6%98%8F%E5%80%92-service-mesh-%E5%8E%9F%E4%BE%86%E4%B8%8D%E6%98%AF%E7%B6%B2%E6%A0%BC%E6%9C%8D%E5%8B%99-9a4b0636371f
