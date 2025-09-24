## K8S Lab Day_17

## 前言

昨天介紹了簡單的 metrics 的用法，那現在我們要來客製化它了～

## Classifying Metrics Based on Request or Response

當我們拿到這些有的沒有的 metrics 一定會覺得很煩，因為大部分是我們不需要的資料，所以我們要將它處理完變成我們喜歡的樣子

### 首先我們要先針對 request 來進行處理

這裡透過 WasmPlugin 的 attributegen 根據 API 路徑與方法產生自訂欄位 `istio_operationId`，再用 Telemetry 把它加到 REQUEST_COUNT metric 的標籤 request_operation，讓 Prometheus 能依不同操作區分流量統計

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin # WasmPlugin 是 Istio 用來載入並運行 WebAssembly 外掛的資源，讓 data plane 中自訂代理的行為
metadata:
  name: istio-attributegen-filter
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  url: https://storage.googleapis.com/istio-build/proxy/attributegen-359dcd3a19f109c50e97517fe6b1e2676e870c4d.wasm
  imagePullPolicy: Always
  phase: AUTHN
  pluginConfig:
    attributes:
      - output_attribute: "istio_operationId"
        match:
          - value: "ListReviews"
            condition: "request.url_path == '/reviews' && request.method == 'GET'"
          - value: "GetReview"
            condition: "request.url_path.matches('^/reviews/[[:alnum:]]*$') && request.method == 'GET'"
          - value: "CreateReview"
            condition: "request.url_path == '/reviews/' && request.method == 'POST'"
---
apiVersion: telemetry.istio.io/v1
kind: Telemetry
metadata:
  name: custom-tags
spec:
  metrics:
    - overrides:
        - match:
            metric: REQUEST_COUNT
            mode: CLIENT_AND_SERVER
          tagOverrides:
            request_operation:
              value: filter_state['wasm.istio_operationId']
      providers:
        - name: prometheus
```

```bash
kubectl -n istio-system apply -f attribute_gen_service.yaml
```

```bash
kubectl exec -it $(kubectl get pod -l app=reviews,version=v1 -o jsonpath='{.items[0].metadata.name}') -- curl localhost:9080/reviews/1
```

## Reference

https://istio.io/latest/docs/tasks/observability/metrics/classify-metrics/
