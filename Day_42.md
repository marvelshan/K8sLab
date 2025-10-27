## K8S Lab Day_42

# Day 4: 當無數個 Goroutine 同時甦醒，我看見了雲原生的靈界之門

## 前言

昨天我們學習了 go module 底層是怎麼去呼叫 function 並使用其方法，那今天我們要來學習 goroutines 與 channels，並對比 k8s 控制器模式的併發需求，也要透過一些小實驗來實作 k8s 的 pod 調度，來了解底層原理的運作，讓我們更清楚 go 的特性對 k8s 和 istio 帶來的好處

## 什麼是 Goroutines？

Goroutine 是 Go 的輕量化執行緒，因為他創建和切換的成本極低，可以輕鬆啟動數千甚至十萬個 goroutine，那要如何使用，在第一天的時候就有使用到，在 k8s 中，deployment controller 需要同時監控多個資源的狀態變化 goroutines 就非常適合來處理這類的任務

```go
go func() {
    fmt.Println("Runnung in a goroutine")
}
```

## 什麼是 Channel？

Channel 是 go 提供於同步和通信的機制，用在 goroutines 之間傳遞資料，channel 卻報資料傳輸的安全，避免 mutex 的複雜性，在第一天也有使用到，在 istio 的 control plane 中，goroutine 可以用來處理多個 sidecar 配置的更新，而 channels 就是用於協調這些更新

```go
ch := make(chan string)
go func() {
    ch <- "Hello from goroutine"
}()
msg := <-ch
fmt.Println(msg)
```

## k8s 中的併發機制

在 k8s 中， controller pattern 依賴併發來確保 cluster 的 actual state 和 desired state 一致，而其中的工作原理是 Deployment Controller、ReplicaSet Controller 透過 reconcile loop 來監控資源的狀態，當檢查到差異時執行動作，像是創建或刪除 Pod，每個 controller 需要併行處理多個資源，且快速地響應事件，K8s controller-runtime 內部廣泛運用這些機制，以保證系統在高頻率事件下仍能穩定運作

## Concurrent Reconciling

在 [k8s controller-runtime](https://github.com/kubernetes-sigs/controller-runtime/blob/main/pkg/controller/controller.go) 中，`MaxConcurrentReconciles` 這個參數決定了同時可以有多少個 reconcile loop 被啟動，當監控的物件頻繁變化時像是大量 Pod 更新，會產生許多 reconcile 請求，如果僅使用單一執行緒處理，reconcile queue 很快就會塞滿，導致延遲累積。此時開啟多執行緒（Concurrent Reconciling）能顯著提升吞吐量

```go
	// MaxConcurrentReconciles is the maximum number of concurrent Reconciles which can be run. Defaults to 1.
	MaxConcurrentReconciles int
...
	// Reconciler reconciles an object
	Reconciler reconcile.TypedReconciler[request]
```

這時候就會想開啟多執行緒後，會不會有兩個 reconcile loop 同時處理同一個物件，造成狀態不一致？

答案是不會！是因為 controller-runtime 內部採用了 k8s client-go 提供的 workqueue 實作，該資料結構在 goroutines 間協調物件的處理狀態

當一個 reconcile request 進入 queue 時，

- 若該物件在 processing set 中正在被處理，僅將它加入 dirty set
- 若該物件尚未被處理，則加入 queue 並標記進入 processing set
- 當該物件處理完畢後，若 dirty set 中仍有它的紀錄，代表在期間內又發生了變化，物件會被重新加入 queue

這樣的設計保證不會同時有兩個 goroutine 處理同一個物件，並能夠在高頻事件中自動合併多次變更，避免重複計算

但這種機制也帶來一個副作用，因為物件可能會在 queue 尾端被重新排入，因此某些請求可能遭遇延遲，特別是長時間的 reconcile 任務中

controller 的行為是 level-triggered，而非 edge-triggered，並不保證會對每一個事件都逐一響應，而是確保系統最終能收斂至正確的狀態，它追求的是狀態一致性，而非事件順序的完整追蹤

## Reference

https://openkruise.io/blog/learning-concurrent-reconciling
