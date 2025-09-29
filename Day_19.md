## K8S Lab Day_19

# Day17: 在 Kubernetes 上使用 Rook 部署 Ceph 作為分散式儲存

## 前言

前幾天陸陸續續地把 istio 的功能都介紹完了，今天來試試個 Ceph，來作為分布式儲存的系統吧

## Ceph

Ceph 是一個開源的分散式儲存系統，目標是提供統一的儲存平台，我的想法比較像是不被 vendor lock-in 的工具啦，主要有三種的存取模式，Object Storage、Block Storage、File System，他主要的運作核心是 RADOS（Reliable, Autonomous, Distributed Object Store），透過 CRUSH 演算法來決定資料放在哪些節點上，並確保多副本或 Erasure Coding，可以達到高可用的效果，Ceph 主要有三個核心的 component

在這張圖 Clients 並不直接操作 OSD，而是透過 Ceph 提供的 Interface Layer 來存取，這一層是 Ceph 對外提供的不同存取方式，RBD (RADOS Block Device)提供區塊儲存，適合掛載給 VM 或 DB，RADOS GW (Gateway)提供 S3 / Swift 相容的物件儲存 API，適合存放圖片、影片、備份等非結構化資料，CephFS (Ceph File System)提供 POSIX 相容的檔案系統，支援多個 Client 共用檔案目錄，Ceph Cluster of Computers，這層是真正執行在多台機器上的 Daemon

| 名稱                            | 英文         | 功能與角色                                                                                                                                     | 部署數量                                                           |
| :------------------------------ | :----------- | :--------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------- |
| **Monitor (MON)**               | Ceph Monitor | 它們是 Ceph 的心臟，負責維護 cluster 的 Cluster Map ，這包含了所有 OSD、MGR 和用戶的狀態。MON 之間透過 Paxos/Raft 等共識演算法來確保狀態一致性 | 建議運行 3 個或 5 個，以達成法定人數（Quorum）來確保高可用性       |
| **Object Storage Daemon (OSD)** | Ceph OSD     | 負責實際儲存資料，處理資料複製、恢復、再平衡。每個 OSD 通常對應到伺服器上的一個 raw block device 或分區                                        | 叢集運行越多 OSD，容量越大，效能也越好                             |
| **Manager (MGR)**               | Ceph Manager | 負責 Ceph 叢集的監控和管理，收集運行狀態指標、提供 API 介面給外部管理工具，例如 Ceph Dashboard                                                 | 建議運行 2 個，一個 Active，一個 Standby，以確保管理服務的高可用性 |

首先我們就來安裝吧，我們所使用到的是 rook，Rook 專門作為在 k8s 上的 Orchestrator，相較於 `ceph-deploy` 和 `ansible` 的做法，Rook 已經把流程都抽象化，直接用 CRDs 來定義 Ceph Cluster 相關的 component，就可以省去很多繁瑣的流程

```bash
git clone --single-branch --branch v1.18.2 https://github.com/rook/rook.git
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f csi-operator.yaml -f operator.yaml
kubectl create -f cluster.yaml
```

其實搭建的步驟蠻簡單明瞭的，接下來我們就可以利用指令來查看是否已經部署完成

```bash
kubectl -n rook-ceph get pod
```

## Reference

https://www.qikqiak.com/k8strain2/

https://medium.com/jacky-life/%E5%9C%A8-k8s-%E4%BD%BF%E7%94%A8-rook-%E5%AE%89%E8%A3%9D-ceph-1999f52a6fb9
