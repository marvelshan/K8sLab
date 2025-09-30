## K8S Lab Day_20

## 前言

接著我們要使用 `Toolbox` 來查看 pod 裡面運作的細節

```bash
kubectl create -f deploy/examples/toolbox.yaml
kubectl -n rook-ceph rollout status deploy/rook-ceph-tools
kubectl -n rook-ceph get pods | grep ceph-tools
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```

接著我們會使用幾個簡單的 ceph 指令查看運作的細節

```bash
ceph status
```

```bash
  cluster:
    id:     1e6f8dde-9de0-4cbe-8104-6d174707a258
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
   # 代表目前有警告，這裡是因為 OSD 數量不足（0 < 3）

  services:
    mon: 3 daemons, quorum a,b,c (age 27h)
    # 三個 monitor 正常
    mgr: b(active, since 27h), standbys: a
    osd: 0 osds: 0 up, 0 in
    # 目前沒有任何 OSD

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

這邊來檢查儲存用量

```bash
ceph df
```

```bash
--- RAW STORAGE ---
CLASS  SIZE  AVAIL  USED  RAW USED  %RAW USED
TOTAL   0 B    0 B   0 B       0 B          0
 # 目前是 0，因為沒有 OSD
--- POOLS ---
# 沒有任何 pool
POOL  ID  PGS  STORED  OBJECTS  USED  %USED  MAX AVAIL
```

還有像是 `ceph osd status` 和 `rados df` 的指令去查詢 ceph 的細節狀態

```bash
kubectl -n rook-ceph logs -l app=rook-ceph-osd-prepare
```

## 調整 Ceph 池（replication / erasure coding）、OSD Crush 規則，做 I/O 壓力測試，模擬 OSD/Node 故障觀察自癒流程

## Reference

https://rook.io/docs/rook/latest-release/Troubleshooting/ceph-toolbox/#interactive-toolbox
