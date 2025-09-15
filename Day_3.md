## K8S Lab Day_3
今天會是貼人賽的第一天，有鑒於現在 ai 盛行，我請他幫我生成了 30 天的計畫，能不能順利地執行完畢是一回事，但是盡力地跟上他的實作吧～

| 天數 | 題目 | 實作重點 |
|------|------|------|
| Day 1 | Kubernetes 簡介：什麼是 K8s？為什麼適合微服務？ | 使用 `Kubespray`，啟動一個本地 K8s 叢集 |
| Day 2 | Kubernetes 核心架構：Master Node 與 Worker Node | 用 `kubectl` 查看叢集狀態，部署一個簡單 Pod |
| Day 3 | Pods 與容器：Kubernetes 的最小部署單位 | 創建一個包含單容器的 Pod，驗證運行狀態 |
| Day 4 | 使用 Nix 設置 Kubernetes 環境：可重現的 Minikube 部署 | 撰寫 `minikube.nix`，用 `nix-shell` 啟動 K8s 環境 |
| Day 5 | 部署與管理：Kubernetes Deployment 的實作 | 部署一個簡單的 Nginx 應用，測試 Rolling Update |
| Day 6 | ConfigMap 與 Secret：管理 K8s 配置 | 用 ConfigMap 配置應用環境變數，實作 Secret |
| Day 7 | Kubernetes 服務發現：Service 與 Ingress 基礎 | 部署一個 Service，測試 ClusterIP 和 NodePort |
| Day 8 | Nix 入門：聲明式包管理的優勢 | 用 Nix 安裝 `kubectl` 和 `helm`，驗證環境一致性 |
| Day 9 | CI/CD 基礎：什麼是 CI/CD？為什麼在雲原生中重要？ | 設置一個簡單的 GitHub Actions 工作流，執行 lint 檢查 |
| Day 10 | 用 Nix 打造 CI/CD 環境：可重現的工具鏈 | 用 Nix 配置 GitHub Actions Runner 環境，安裝依賴 |
| Day 11 | Service Mesh 入門：從 Kubernetes 到 Service Mesh | 部署一個簡單的微服務應用，觀察通信挑戰 |
| Day 12 | Service Mesh 架構：Data Plane 與 Control Plane | 安裝 Istio（用 Nix 配置），驗證 Sidecar 注入 |
| Day 13 | 使用 CI/CD 部署 Istio：自動化 Service Mesh 設置 | 撰寫 GitHub Actions 工作流，自動部署 Istio 到 K8s |
| Day 14 | Service Mesh 中的服務發現：自動化服務註冊 | 配置 Istio 服務發現，測試服務間通信 |
| Day 15 | 流量管理基礎：Istio 的 Routing Rules 實作 | 設置 VirtualService，實現簡單的路由規則 |
| Day 16 | 負載平衡進階：Service Mesh 中的負載平衡策略 | 配置 DestinationRule，測試輪詢負載平衡 |
| Day 17 | 故障注入實戰：模擬 Service Mesh 中的故障 | 用 Istio 注入延遲或錯誤，觀察應用行為 |
| Day 18 | Service Mesh 安全性：mTLS 加密通信 | 啟用 Istio 的 mTLS，驗證服務間加密通信 |
| Day 19 | CI/CD 與 Service Mesh：自動化 Canary Release | 用 GitHub Actions 部署 Canary 版本，結合 Istio 路由 |
| Day 20 | 可觀測性基礎：Service Mesh 中的 Metrics 和 Logs | 安裝 Prometheus（用 Nix），收集 Istio Metrics |
| Day 21 | 分佈式追蹤：Jaeger 在 Service Mesh 中的應用 | 部署 Jaeger，追蹤微服務請求路徑 |
| Day 22 | 用 Nix 管理監控工具：聲明式 Prometheus 部署 | 撰寫 Nix 配置，部署 Prometheus 和 Grafana |
| Day 23 | Circuit Breaking：Service Mesh 中的斷路器模式 | 配置 Istio 的斷路器，測試服務保護機制 |
| Day 24 | Rate Limiting：Service Mesh 中的流量控制 | 設置 Istio 的 Rate Limit，模擬流量限制 |
| Day 25 | 多叢集 Service Mesh：跨 K8s 叢集的部署 | 用 Nix 配置多叢集環境，測試 Istio 跨叢集通信 |
| Day 26 | Service Mesh 與 API Gateway：整合 Envoy | 部署 Envoy 作為 API Gateway，與 Istio 結合 |
| Day 27 | CI/CD 進階：自動化 Service Mesh 測試與部署 | 用 GitHub Actions 實現 Service Mesh 配置的自動測試 |
| Day 28 | 問題排查：Debugging K8s、Nix 和 Service Mesh | 實作排查 Istio 配置錯誤，分享 debug 技巧 |
| Day 29 | 學習心得：從 K8s、Nix 到 Service Mesh 的成長 | 分享實作中的挑戰與解決方法 |
| Day 30 | 總結與展望：Nix、Service Mesh 和 CI/CD 的未來 | 部署一個完整微服務項目，展示成果 |

---

### 什麼是 K8S？
首先先來介紹什麼事 K8S 就是在 Kubernetes 中間有8個英文字母，未來會接觸到更多的單字，像是可觀測性 Observability 又可簡稱o11y，那為何是 Kubernetes 這個單字呢？這個單字是來自希臘文，意思為「舵手，船長」，也是代表了這個工具管理了容器的部署，擴展等等的應用，就像是一位船長必須掌控著整艘船的運作正常。

那我們再回頭來講，會和需要管理這些容器，假如今天的應用場景是單容器，那我們簡單的使用 docker 去 build 一個 image，直接進行部署，這樣好像就不太會使用到，但今天假如服務慢慢地擴大，我們需要更多的容器，更加複雜的網路管理，這樣我們一個一個容器去做部署，好像就不那麼直覺，直接開機器開到死，當然這只是 k8s 其中一個優勢，還有他可以設定自動擴展，高可用的狀況等等。

那再回來繼續介紹 k8s，引用 document 裡面所說：
> Kubernetes is a portable, extensible, open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

他是一個便於跨平台跨環境，可擴充的開源管理容器化服務的平台，他支援宣告式的配置，他是一個大且快速成長的生態系，並且有相當多的服務和支援的工具。

聽起來相當的不賴對吧，那接著要來說明為服務這個詞了。

### 為什麼適合微服務？
首先我們要先來認識微服務 microservices 這個詞，一個來看 document
> Microservices - also known as the microservice architecture - is an architectural style that structures an application as a collection of two or more services that are:
> - Independently deployable
> - Loosely coupled
>
> Services are typically organized around business capabilities. Each service is often owned by a single, small team.
<img width="1019" height="443" alt="截圖 2025-09-15 下午1 33 34" src="https://github.com/user-attachments/assets/0d342f62-9ef1-4b76-904b-3b0e507c446c" />

是一種架構風格，它將應用程式設計為由兩個或更多服務所組成的集合，服務具有以下特性，可獨立部署，低耦合，服務通常是依照業務能力來劃分，每個服務通常由一個小型團隊負責。

看完文件，好像也沒比較好懂，會拿來比較的就會是單體式服務(Monolithic)，這種服務的架構設計相對單一，功能服務全部都在一個 service 裡面，不用思考他會跟其他的服務交互的運作，但缺點就是當服務越加龐大，scaling 的成本就會愈加困難，對於單一語言的依賴性就會更強。這時候就可以回來介紹微服務，各個功能會拆開不同的 service，像是一個售票系統，會員登入會是一個 service，結帳會是一個 service 等等，這樣未來我們再針對於服務的 bottleneck 時，我們就可以針對性的 scaling 單一服務，而不是整個服務一同 scaling，降低了擴展的成本，這個過程也產生相對應的缺點，整體的服務間相互溝通，當出錯時就會需要考慮多個服務的運作是哪裡出問題。

所以看完這些介紹，微服務就一定是最好的嗎？不一定，一切取決於當下的應用場景與未來的架構考量。

回到 k8s，有提到說當今天的大量的容器需要被開啟，那不是就跟微服務的架構不謀而合嗎～把不同的功能分為不同的微服務，並利用 k8s 的優勢去啟動和管理這些服務不是更加容易嗎～並且其中有很好的網路通信機制，讓我們能更佳容易的去管理這些微服務。

### 使用 `Kubespray`，啟動一個本地 K8s 叢集
接著就是開始實作的部分，我的環境是參考 Tico 大大的[文章](https://ithelp.ithome.com.tw/users/20112934/ironman/5640)去架設的，非常推薦觀看～
大部分都是按照大大的實作，我這邊就不多做說明了，因為在運行完以下指令可能會多花一點時間。

這邊介紹幾個指令：
```shell
test -f requirements-$ANSIBLE_VERSION.yml && \
ansible-galaxy role install -r requirements-$ANSIBLE_VERSION.yml && \
ansible-galaxy collection install -r requirements-$ANSIBLE_VERSION.yml
```
- 目的：Kubespray 是用 Ansible 部署 Kubernetes，所以需要依賴一些 Ansible roles 與 collections，最後確保部署環境有正確的依賴，避免 playbook 執行失敗。

- 步驟：
1. test -f requirements-$ANSIBLE_VERSION.yml：檢查該版本的 requirements.yml 是否存在，如果不存在就不執行後續指令。

2. ansible-galaxy role install -r requirements-$ANSIBLE_VERSION.yml：從 requirements.yml 安裝所有指定的 Ansible roles。

3. ansible-galaxy collection install -r requirements-$ANSIBLE_VERSION.yml：安裝 Ansible collections（可能包含 module、plugin 等）。
```shell
vi inventory/mycluster/inventory.ini
```
目的：指定你自己 Kubernetes cluster 的節點資訊和角色。
```yaml
[all]
k8s-m0 ansible_host=192.168.200.126 ansible_user=ubuntu
k8s-n0 ansible_host=192.168.200.54 ansible_user=ubuntu
k8s-n1 ansible_host=192.168.200.249 ansible_user=ubuntu
...
```
接著就是執行命令去安裝了！
```shell
ansible-playbook -i inventory/mycluster/inventory.ini --private-key=~/private.key --become --become-user=root cluster.yml
```

結果：
```
PLAY RECAP *********************************************************************
k8s-m0                     : ok=31   changed=3    unreachable=0    failed=0    skipped=58   rescued=0    ignored=0   
k8s-n0                     : ok=26   changed=3    unreachable=0    failed=0    skipped=48   rescued=0    ignored=0   
k8s-n1                     : ok=25   changed=3    unreachable=0    failed=1    skipped=48   rescued=0    ignored=0   
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
因為我在開機器的時候因為 Quota 不夠，在 k8s-n1 這台機器只開啟了 d1.tiny 的機器，顯然 ram 只有 512MB 是不太夠的

目前的做法我是去調整`minimal_node_memory_mb`的限制，讓他降至512MB，假如未來有問題再來調整吧～
```
grep -R "minimal_node_memory_mb" inventory/mycluster/group_vars/ roles/kubernetes/preinstall/defaults/
roles/kubernetes/preinstall/defaults/main.yml:minimal_node_memory_mb: 1024
```
假如沒有顯示 error 就是成功安裝完畢啦！（目前還是失敗QQ）
