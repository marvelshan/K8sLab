## K8S Lab Day_12

## 前言

昨天做了 istio 比較基本可以做到的功能認證和授權，接著就是比較應用層面的，今天會講到比較應用層面的場景，在微服務架構下，某個服務要去呼叫別的服務是一件很常見的事，但問題是如果下游服務反應很慢，甚至卡住不回應，我上游的服務要等多久才算「超時」？ 這就是我們今天要透過 Istio 來解決的流量的 Timeout 控制

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

那文件中有 Request Routing，Fault Injection，Traffic Shifting，TCP Traffic Shifting，Request Timeouts，Circuit Breaking，Mirroring，Locality Load Balancing，Ingress，Egress 這些的場景，我會拿幾個來實作一下

這邊因為需要做到內網的連線，所以我也參考了 [metalb](https://ithelp.ithome.com.tw/articles/10301327) 和使用 [helm 去做 ingressgateway](https://ithelp.ithome.com.tw/articles/10301329) 來讓內網進行連線，這樣我們就不用像官網再去 `kubectl get crd` 這是用來安裝 Kubernetes Gateway API 的 CRDs (Gateway, HTTPRoute 等)，那你一定很好奇這兩種有什麼差異呢？

那我們來做一個小實驗吧，是 Request Routing + Traffic Shifting → A/B 測試、Canary

1. 首先我們要先部署一個簡單的 curl Pod，讓可以去測試流量

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl
  namespace: default
  labels:
    app: curl
spec:
  containers:
    - name: curl
      image: curlimages/curl:8.8.0
      command: ["sleep", "3600"]
```

```bash
kubectl apply -f curl.yaml
```

2. Deploy 我們需要測試的 sample bookinfo（這些有 sample 的指令必須要在昨天 clone 下來的 istio/ 下會找到這些檔案喔）

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl label namespace default istio-injection=enabled
```

在建立的時候會出現 v3 的版本，因為我目前測試不到，可以先只留下 v1 和 v2

```bash
kubectl delete deployment -l app=reviews,version!=v1,version!=v2 -n default
```

3. 確認服務已經都啟動

```bash
kubectl get pods
kubectl get svc
```

4. 建立一個 DestinationRule 和 VirtualService，先把 100% 流量導到 v1

```bash
kubectl apply -f - <<EOF
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
EOF
```

```bash
kubectl apply -f - <<EOF
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
      weight: 100
EOF
```

5. 做 Canary，上線 v2，流量 90/10

```bash
kubectl apply -f - <<EOF
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
EOF
```

6. 測試看看流量分佈

```
kubectl exec -n default   -it $(kubectl get pod -n default -l app=curl -o jsonpath={.items[0].metadata.name})   -c curl -- curl -s -D - http://productpage:9080/productpage | grep "review"
```

## Reference

https://ithelp.ithome.com.tw/articles/10301327

https://ithelp.ithome.com.tw/articles/10301329

https://medium.com/brobridge/%E6%98%8F%E5%80%92-service-mesh-%E5%8E%9F%E4%BE%86%E4%B8%8D%E6%98%AF%E7%B6%B2%E6%A0%BC%E6%9C%8D%E5%8B%99-9a4b0636371f
