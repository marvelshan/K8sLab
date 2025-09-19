## K8S Lab Day_3
今天會是貼人賽的第一天，有鑒於現在 ai 盛行，我請他幫我生成了 30 天的計畫，能不能順利地執行完畢是一回事，但是盡力地跟上他的實作吧～

| 天數 | 題目 | 實作重點 |
|------|------|------|
| Day 1 | 認識 Kubernetes 與微服務 | 使用 `Kubespray`，啟動一個本地 K8s 叢集 |
| Day 2 | Master Node 與 Worker Node| 用 `kubespray` 查看叢集狀態，部署一個簡單 Pod |
| Day 3 | Kubernetes 核心資源：服務、命名空間、儲存與配置管理 | --- |
| Day 4 | 使用 Nix 打造可重現的 Kubespray Kubernetes 叢集 | 使用 Nix 管理 Kubespray 相關配置可重現 |
| Day 5 | Nix + Flake 你的開發環境 01 號機 | 用 `flake.nix` 來了解實際運作 |
| Day 6  | Pod 調度與資源管理（Requests / Limits / QoS）           | 為 Pod 設定 CPU/Memory requests & limits，觀察 Scheduler 行為，模擬資源飽和與預留（LimitRange, ResourceQuota）。                            |
| Day 7  | StatefulSet 與 PersistentVolume 基礎              | 部署一個使用 PVC 的 StatefulSet（例如 MySQL），了解 PVC、PV、StorageClass 的聯動與綁定流程。                                                    |
| Day 8  | 用 Nix 管理 Kubernetes manifests（Helm + Nix）      | 把你前面部署的 manifests 用 Nix 包裝（flake），示範如何用 Nix build 自動產生 YAML 並部署。                                                       |
| Day 9 | GitLab 基礎與 Runner 設置                           | 在本地或 VM 安裝 GitLab CE（或使用 GitLab.com），註冊一個 Docker runner（或 k8s runner），測試簡單 pipeline（build → echo）。                     |
| Day 10 | 用 GitLab CI + Nix 建置可重現的 CI Job                | 撰寫 `.gitlab-ci.yml`，在 pipeline 中使用 Nix（或 shell wrapper）執行 `nix build`、跑單元測試、上傳 artifact。                               |
| Day 11 | 存儲策略規劃：Ceph 基礎概念與選型（RBD/CephFS/RGW）            | 依需求決定 RBD、CephFS、RGW 的用途；寫下容量、PVC 類型、故障域（failure domain）與 CRUSH 規劃草案。                                                  |
| Day 12 | 在 Kubernetes 上部署 Rook Operator（Ceph）           | 使用 Rook Operator 部署 Ceph Cluster（最小 3 節點），觀察 ceph status，熟悉 CephCluster CRD。                                           |
| Day 13 | 建立 StorageClass 與測試 PVC（RBD / CephFS）          | 建 RBD 與 CephFS 的 StorageClass，申請 PVC 並掛載到 Pod，測試讀寫與故障恢復（模擬 OSD 下線）。                                                    |
| Day 14 | RGW（S3 endpoint）部署與對象存儲測試                      | 啟動 RGW，使用 s3cmd 或 awscli 測試上傳/下載，做為 artifact / dataset 存放示範。                                                           |
| Day 15 | Service Mesh 入門：選型與基本安裝（Istio 或 Linkerd）       | 選擇 Service Mesh（建議先以 Istio 練習），在 K8s 安裝控制平面，部署示範 app 並啟用 sidecar，觀察流量。                                                 |
| Day 16 | Mesh 基本流量控制：mTLS、Policy、Timeout、Retry          | 啟用 mTLS，實作流量限速、timeout 與 retry 規則，驗證失敗重試與熔斷行為。                                                                         |
| Day 17 | Canary / A/B 測試實作（流量分割）                        | 用 Service Mesh 做 Canary（例：10% 流量到 v2），撰寫示例配置並驗證流量比例與指標收集。                                                              |
| Day 18 | 多模型 / 多路由場景：條件路由與灰度策略                          | 實作 header 或路徑基礎的路由策略（小模型 vs 大模型），模擬成本節省策略（大多數請求用小模型）。                                                                  |
| Day 19 | CI/CD 與 Mesh 整合：在 pipeline 中自動化 Canary 部署      | 實作 GitLab CI job：build image → push → 更新 manifest，pipeline 執行時觸發 Canary 流量設定（或更新 Service 相關註解）。                        |
| Day 20 | 觀測性（Metrics）設置：Prometheus + Grafana            | 部署 Prometheus Operator，抓取 K8s / Ceph / Mesh / app 指標，建立 Grafana Dashboard（GPU、Pod、Ceph IOPS）。                          |
| Day 21 | Logging 與 Tracing：EFK 或 Loki + Jaeger          | 部署 Fluentd/Fluent Bit 或 Loki 收集日誌，部署 Jaeger（或 OpenTelemetry Collector）做分布式追蹤，串接 Mesh 提供的 tracing header。               |
| Day 22 | Ceph 深入：性能調校與故障模擬                              | 調整 Ceph 池（replication / erasure coding）、OSD Crush 規則，做 I/O 壓力測試，模擬 OSD/Node 故障觀察自癒流程。                                  |
| Day 23 | 存儲快照與備份：Rook Ceph 與外部備援                        | 設定 Ceph snapshot，測試 PVC snapshot 與 restore，整合對象存儲做異地備援（例如 RGW -> 外部 S3 / 本地備份）。                                        |
| Day 24 | 安全性硬化：K8s、Ceph、GitLab、Mesh 的安全實作               | 為 K8s 啟用 RBAC、NetworkPolicy；Ceph 確認 keyring 與用戶權限；GitLab 設定 MFA & protected branches；Mesh 設定授權策略（AuthorizationPolicy）。 |
| Day 25 | 自動化擴充：HPA / VPA 與 Ceph 容量擴容流程                  | 設定 Horizontal Pod Autoscaler、Vertical Pod Autoscaler，模擬流量增加並觀察自動擴縮；練習擴充 Ceph OSD 節點並觀察 rebalance。                      |
| Day 26 | 性能測試與壓力測試（推理場景）                                | 使用工具（wrk、hey、locust 或自製 workload）壓測推理服務，觀察延遲分布、sidecar 開銷、Ceph I/O 瓶頸。                                                 |
| Day 27 | Chaos Engineering：模擬失敗與韌性演練                    | 使用 Chaos Mesh / LitmusChaos 做故障注入（Pod crash、node loss、network partition），驗證系統可用性與自癒能力。                                 |
| Day 28 | MLOps 與模型管理：Artifact、Checkpoint、Registry       | 用 Ceph RGW 做模型 artifact 存放，設計模型版本命名與檢索流程，CI pipeline 自動上傳訓練 checkpoint 至 object store。                                 |
| Day 29 | 最後整合總驗收（End-to-End）與文件化                        | 做一次從 code -> CI -> build -> deploy -> Canary -> promote -> rollback 的完整驗收演練，產出系統設計文件、運維 runbook 與一份 30 天學習總結。          |


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
因為我在開機器的時候因為 Quota 不夠，在 k8s-n1 這台機器只開啟了 d1.tiny 的機器，顯然 ram 只有 512MB 是不太夠的，這邊的問題是可以用以下指令去尋找說 `minimal_node_memory_mb` 這裡的最低 memory 是 1024MB 像我所設定的 512MB 顯然時不足夠的，這邊就要去調整 ram 的大小去符合他，調整完就可以順利啟動啦！
```
grep -R "minimal_node_memory_mb" inventory/mycluster/group_vars/ roles/kubernetes/preinstall/defaults/
roles/kubernetes/preinstall/defaults/main.yml:minimal_node_memory_mb: 1024
```
