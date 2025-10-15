## K8S Lab Day_33

# Pod 生命週期

<img width="1229" height="1051" alt="image" src="https://github.com/user-attachments/assets/f72dbd23-0cf5-4572-ab9b-8d2146420507" />

pod 的生命週期是從他被排程到節點上開始直到被終止為止的過程，這個過程是被嚴格按照順序進行，必且確保都是處於正確的狀態

## 1. Init Container

這個階段主要是在啟動之前所有的前置工作，主要是要設定配置的檔案，檢查有沒有必要的服務，並執行資料庫的遷移，或是 istio sidecar 等等需要的 injection

## 2. Pod Hook

當 init 完成後，main container 會開始啟動，這是會觸發 pod hook，在主容器的 Entry Point 執行後立即觸發，但不需要等待主容器內的應用程式啟動完畢

### 3. Probes

在 main container 進入運行狀態後，會透過兩種 probes 來監控他們的健康狀態，第一是 readiness probe 用來判斷應用程式是否已經準備好接受流量，再來是 liveness peobe 用來判斷 container 是否仍存活著，是否需要 restart

# Istio 與 Gateway API Inference Extension：為 Kubernetes 上的 AI 推理打造智慧流量路由

## 前言

在 k8s 運行 ai workload 一直都是一個很大的挑戰，在我昨天所提到的 ai 會需要判斷他的資源使用率才能達到最好的成本考量，也有提到因為他的推理請求每個都相當的不同，像是他的記憶體消耗量大，有時候會需要使用到動態載入 Lora Low-Rank Adapter，在平常的服務，我們常常使用 ingres、gateway api、service mesh 等等的 L7 的流量控制，但這些往往對 ai 不理想，所以 Gateway API Inference Extension 是要來解決這個問題，讓 gateway 能理解 ai 的推理，然後比較聰明的方式去使用 GPU 的資源

## Gateway API Inference Extension 的設計

<img width="1298" height="602" alt="截圖 2025-10-15 下午3 10 50" src="https://github.com/user-attachments/assets/2c5681c7-1e34-43e0-9f5d-27af8d261ec0" />

### 1. InferenceModel

這裏主要是由 ai 工程師來決定，這是邏輯模型的入口，支持多個模型來去做切分

```yaml
apiVersion: inference.networking.x-k8s.io/v1alpha2
kind: InferenceModel
metadata:
  name: inferencemodel-llama2
spec:
  modelName: llama2
  criticality: Critical
  poolRef:
    name: vllm-llama2-7b-pool
  targetModels:
    - name: vllm-llama2-7b-2024-11-20
      weight: 75
    - name: vllm-llama2-7b-2025-03-24
      weight: 25
```

### 2. InferencePool

這裏主要是由維運人員來去做配置，HTTPRoute 可以將流量導向一個智能的推理池，也就是這裏所説到的 `InferencePool`

```yaml
apiVersion: inference.networking.x-k8s.io/v1alpha2
kind: InferencePool
metadata:
  name: vllm-llama2-7b-pool
spec:
  targetPortNumber: 8000
  selector:
    app: vllm-llama2-7b
  extensionRef:
    name: vllm-llama2-7b-endpoint-picker
```

### 3. HTTPRoute

這裏就從剛剛的推理入口送到了 InferencePool 的 HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: llm-route
spec:
  parentRefs:
    - name: inference-gateway
  rules:
    - backendRefs:
        - group: inference.networking.x-k8s.io
          kind: InferencePool
          name: vllm-llama2-7b
      matches:
        - path:
            type: PathPrefix
            value: /
```

## Reference

https://cloud.tencent.com/developer/article/2522826

https://istio.io/latest/zh/blog/2025/inference-extension-support/

https://www.solo.io/blog/llm-d-distributed-inference-serving-on-kubernetes

https://cloudnativecn.com/blog/gateway-api-inference-extension-deep-dive/
