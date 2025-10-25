## K8S Lab Day_40

# 用桑原的靈氣之劍剖析開源專案的結構

## 前言

上篇講到如何使用 go 來建立一個簡單的 HTTP server，今天會來探索 k8s 的結構，透過 go 來模擬 kubectl 的 CLI 工具，那 go 的設計理念又是什麼呢？我們來看看[官方文件](https://go.dev/talks/2012/splash.article?utm_source=chatgpt.com)怎麼說

> Go’s purpose is therefore not to do research into programming-language design; it is to improve the working environment for its designers and their coworkers.

目的在於提升團隊開發效率與大型軟體可維護性，而不是單純進行程式語言設計的研究，這樣我們就可以了解到 go 想要創造一個可讀性高、維護性強、開發速度快的 programming environment，讓 developer 可以更專注於解決實際的問題，而不是限於語法糖和複雜的語法設計，這就是為什麼 GO 如此的強調簡潔而實用的哲學，沒有複雜的繼承關係、過多的語法糖，而是用 struct 和 interface 與一致的 gofmt 來讓團隊協作更加的容易

## 探索 Kubernetes repo 的程式結構

k8s 的 source code 是一個學習 Go 和 cloud native 設計的寶庫，它的程式碼結構清晰，也遵循 Golang 的慣例

`cmd/` 存放可執行程式的進入點，像是 `cmd/kube-apiserver` 是 API server 的主要程式，`cmd/kubectl` 是 CLI 工具的入口，這種檔案通常很短，主要負責啟動和配置

`pkg/` 存放核心的邏輯和共用的 code，像是 `pkg/api` 定義了 API 物件像是 Pod、Service，`pkg/controller` 實現了 controller 的邏輯像是 Deployment，`pkg/scheduler` 包含了 scheduletr 相關的 code

## 小實驗

### 用 Go 撰寫一個簡單的 CLI 工具

假如我們要實作一個 kubectl 工具，並列出 Pod，我們要透過 struct 來模擬 Pod，並透過命令參數來處理 list

首先我們要先初始化 Go module

```bash
go mod init simple-kubectl
vi main.go
```

```go
package main

import (
    "flag"
    "fmt"
    "log"
)

type Pod struct {
    Name    string
    Namespace string
    Status string
}

// struct 用於定義複雜資料結構

func ListPods() ([]Pod, error) {
    // 模擬資料，實際上應從 API Server 獲取
    pods := []Pod{
        {Name: "pod-1", Namespace: "default", Status: "Running"},
        {Name: "pod-2", Namespace: "default", Status: "Pending"},
    }
    log.Println("Listing pods...")
    return pods, nil
}

// 這裏定義了一個 ListPods function，用來模擬列出 k8s 中的 Pod list，實際情況下應該從 API Server 取得資料，但這裡以兩個範例 Pod 模擬回傳，並在執行時輸出日誌「Listing pods...」，最後返回 Pod 陣列與 nil 錯誤值

func PrintPods(pods []Pod) {
    for _, pod := range pods {
        fmt.Printf("Pod: %s/%s, Status: %s\n", pod.Namespace, pod.Name, pod.Status)
    }
}
// 這裏定義了 PrintPods function，用於將傳入的 Pod 清單逐一輸出到 terminal，它透過 for loop 遍歷 pods 陣列，並使用 fmt.Printf 以「命名空間/Pod 名稱」及「狀態」的格式列印出每個 Pod 的資訊，方便觀察目前叢集中的 Pod 狀態

func main() {
    // 設定日誌前綴與格式
    log.SetPrefix("simple-kubectl: ")
    log.SetFlags(log.LstdFlags | log.Lshortfile)

    // 解析命令列參數
    command := flag.String("command", "", "Command to execute (e.g., list)")
    flag.Parse()

    if *command == "list" {
        pods, err := ListPods()
        if err != nil {
            log.Fatalf("Failed to list pods: %v", err)
        }
        PrintPods(pods)
    } else {
        log.Println("Unknown command. Use --command=list to list pods.")
        flag.Usage()
    }
}
```

接著執行

```bash
go run main.go --command=list
```

```bash
# Output:
simple-kubectl: 2025/10/25 11:44:38 main.go:22: Listing pods...
Pod: default/pod-1, Status: Running
Pod: default/pod-2, Status: Pending
```

```bash
go run main.go --command=invalid
```

```bash
# Output
simple-kubectl: 2025/10/25 11:44:48 main.go:49: Unknown command. Use --command=list to list pods.
Usage of /home/ubuntu/.cache/go-build/21/21356fd3209f2241b737ff1f79446d492772592f6c34946d70eebfce2f757918-d/main:
  -command string
    	Command to execute (e.g., list)
```

接著我們就是要來實際操作來去訪問我們現在的 pod 啦，也就是功能 `kubectl get pods`，但是首先環境已經有 k8s cluster

```bash
go mod init simple-kubectl
```

```go
package main

import (
    "context"
    "flag"
    "fmt"
    "log"
    "os"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

// 這段 import 區塊匯入了 Go library 與 k8s client-go 套件，context 用於控制請求生命週期、flag 解析命令列參數、fmt 與 log 負責輸出與日誌、os 提供系統層操作，metav1 是 k8s 物件的共用中繼資料結構、k8s 提供與叢集互動的主客戶端、clientcmd 則負責載入 kubeconfig 檔案，用於建立連線設定

type Pod struct {
	Name      string
	Namespace string
	Status    string
}

type Lister interface {
	List(namespace string) ([]Pod, error)
}

type K8sClient struct {
	clientset *kubernetes.Clientset
}

func (c *K8sClient) List(namespace string) ([]Pod, error) {
	log.Printf("Listing pods in namespace %s", namespace)
	pods, err := c.clientset.CoreV1().Pods(namespace).List(context.TODO(), metav1.ListOptions{})
	if err != nil {
		log.Printf("Error listing pods: %v", err)
		return nil, err
	}

	var result []Pod
	for _, pod := range pods.Items {
		result = append(result, Pod{
			Name:      pod.Name,
			Namespace: pod.Namespace,
			Status:    string(pod.Status.Phase),
		})
	}
	log.Printf("Found %d pods in namespace %s", len(result), namespace)
	return result, nil
}

// 這裏定義了 K8sClient 的 List 方法，用於列出指定 namespace 的 k8s Pod，它透過 log.Printf 輸出正在列出 Pod 的訊息，接著使用 client-go 的 CoreV1().Pods(namespace).List(...) 呼叫 k8s API Server 取得該 ns 的 Pod list，若 API 調用失敗，會記錄錯誤並回傳 nil 與錯誤值，程式透過迴圈遍歷每個 Pod，將其名稱、命名空間與狀態（pod.Status.Phase）封裝成自定義的 Pod 結構，並加入結果陣列中

func PrintPods(pods []Pod) {
	for _, pod := range pods {
		fmt.Printf("Pod: %s/%s, Status: %s\n", pod.Namespace, pod.Name, pod.Status)
	}
}

func main() {
	log.SetPrefix("simple-kubectl: ")
	log.SetFlags(log.LstdFlags | log.Lshortfile)

    // 這裏設定了 CLI 工具的日誌與命令列參數，並建立與 k8s cluster 的連線，透過 log.SetPrefix 與 log.SetFlags 設定日誌前綴與顯示格式，包括時間戳與程式碼檔案位置

	command := flag.String("command", "", "Command to execute (e.g., list)")
	namespace := flag.String("namespace", "default", "Kubernetes namespace to query")
	kubeconfig := flag.String("kubeconfig", "", "Path to kubeconfig file (default: ~/.kube/config)")
	flag.Parse()

    // 接著使用 flag 定義三個命令列參數：command 指定要執行的操作（如 list）、namespace 指定查詢的 Kubernetes 命名空間（預設 default）、kubeconfig 指定 kubeconfig 檔案路徑（預設為 ~/.kube/config）

	if *kubeconfig == "" {
		home, _ := os.UserHomeDir()
		*kubeconfig = home + "/.kube/config"
	}
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	if err != nil {
		log.Fatalf("Failed to load kubeconfig: %v", err)
	}

	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		log.Fatalf("Failed to create Kubernetes client: %v", err)
	}
	client := &K8sClient{clientset: clientset}

    // 解析參數後，如果未指定 kubeconfig，程式會自動從使用者主目錄載入預設路徑，再透過 clientcmd.BuildConfigFromFlags 讀取 kubeconfig 並建立配置，接著用 kubernetes.NewForConfig 建立 Kubernetes 客戶端 clientset，最後將其封裝到自訂的 K8sClient 結構中，方便後續呼叫 List 或其他方法與叢集互動

	if *command == "list" {
		pods, err := client.List(*namespace)
		if err != nil {
			log.Fatalf("Failed to list pods: %v", err)
		}
		PrintPods(pods)
	} else {
		log.Println("Unknown command. Use --command=list to list pods.")
		flag.Usage()
	}
}
```

接著執行這個指令，這個指令是 go module 管理的指令，用來分析專案的匯入路徑，新增缺少的 dependency 和移除沒有使用的 package，他會同步 `go.mod` 和 `go.sum`，通常都是在有新增 import 之後執行

```bash
go mod tidy
```

然後就可以獲取我們想要的資料啦！

```bash
go run main.go --command=list --namespace=default

# Output
simple-kubectl: 2025/10/25 12:02:26 main.go:34: Listing pods in namespace default
simple-kubectl: 2025/10/25 12:02:26 main.go:49: Found 9 pods in namespace default
Pod: default/curl-74c989df8d-h4jcn, Status: Running
Pod: default/details-v1-766844796b-zj9mg, Status: Running
Pod: default/httpbin-686d6fc899-44csh, Status: Running
Pod: default/productpage-v1-54bb874995-77p7m, Status: Running
Pod: default/ratings-v1-5dc79b6bcd-7cn6g, Status: Running
Pod: default/reviews-v1-598b896c9d-2sbb2, Status: Running
Pod: default/reviews-v2-556d6457d-mc2rq, Status: Running
Pod: default/reviews-v3-564544b4d6-bcfdv, Status: Running
```

當然假如想要獲取更多的資料就可以在 List 方法中增加額外的輸出，第一種是直接印出整個 Pod 物件，使用 `fmt.Printf("%+v\n", pod)`，可以快速看到 Pod 所有欄位與值，適合用於除錯或探索未知欄位，但就是多了點資料 XD，第二種則是選擇性印出重要欄位，例如 Pod 的 ns、名稱、Phase、Node、Pod IP、建立時間以及 container 資訊，這種方式讓輸出更清楚易讀，方便快速掌握叢集中 Pod 的運行狀態與部署細節

```go
func (c *K8sClient) List(name string) ([]Pod, error) {
    log.Printf("Listing pods in namespace %s", namespace)
	pods, err := c.clientset.CoreV1().Pods(namespace).List(context.TODO(), metav1.ListOptions{})
	if err != nil {
		log.Printf("Error listing pods: %v", err)
		return nil, err
	}

	var result []Pod

    // 這邊加上想要輸出的東西，假如想要印出所有東西的話可以試試看以下
    for _, pod := range pods.Items {
        fmt.Printf("%+v\n", pod)
    }

    // 這邊我就印出我想要看到的東西
    for _, pod := range pods.Items {
        fmt.Printf(
            "Pod: %s/%s\n"+
                "  Status: %s\n"+
                "  Node: %s\n"+
                "  Pod IP: %s\n"+
                "  Start Time: %v\n"+
                "  Containers:\n",
            pod.Namespace,
            pod.Name,
            pod.Status.Phase,
            pod.Spec.NodeName,
            pod.Status.PodIP,
            pod.Status.StartTime,
        )
    }
}
```

## struct 與 interface

也許大家對這兩個已經分得很清楚了，但我在一開始看到的時候還是有些搞混，這邊來介紹一下

### struct

是用來定義資料結構，可包含多個欄位與其型別，是「值的容器」，用於儲存資料或狀態

```go
type Pod struct {
	Name      string
	Namespace string
	Status    string
}
```

### interface

是用來定義行為或方法的集合，不儲存資料，描述「任何實作了這些方法的型別，都可以被視為這個介面」

```go
type Lister interface {
	List(namespace string) ([]Pod, error)
}
```

Lister interface 定義了列出 Pod 的行為，任何 struct 只要實作了 `List(namespace string) ([]Pod, error)` 方法，就自動符合這個介面

```go
type K8sClient struct {
	clientset *kubernetes.Clientset
}

func (c *K8sClient) List(namespace string) ([]Pod, error) { ... }
```

這樣 K8sClient 就可以當作 Lister 使用，讓程式可以對`列出 Pod 的行為`進行抽象，而不用依賴具體實作

## 小結

今天學了 struct 和 interface 這個知識點和 kubectl 的運作，雖然還是有些不熟悉語法的操作，但只要多使用一定能更加的熟悉，還有看到 k8s source code `cmd/` 和 `pkg/` 的結構，利用 k8s library 的方法拿出 pod 的資料，這樣好像又離 k8s source code 近了一些～

## Reference

https://cloud.tencent.com/developer/article/1399238

https://go.dev/doc/effective_go

https://go.dev/doc/comment

https://go.dev/play/

https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/kubectl/pkg/cmd/get/get.go
