## K8S Lab Day_37

# Docker 網路實戰：從容器互聯到 Bridge 與 Host 模式解析

## 前言

昨天在 linux 的練工坊讓自己好像馬步紮了比較穩了，但是還是要常常練習把他刻在腦袋裡面，可以時不時就 nslookup、dig、netstat 一下，多敲一下 google.com 或 yahoo.tw 來練習一下 XD，話說從 istio 到現在的 linux 排解，很大一部分都是網路的問題，其實在容器和我們所使用的軟體的世界很大一部分都是跟網路有關，像是最近 XD 就是最近，AWS 在前天大當機，好像是 Dynamo DB 的 API 的 DNS 解析失敗，造成一系列的 Cascade 效應，導致一堆服務死光光，而且還停機停超久，超級 GG，所以這也告訴我們網路是相當重要的一個角色，今天我們要來了解一下 docker 裡面網路的，也就是容器之間的連接

## 為什麼需要容器網路？

在 Docker 的世界裡，容器就像獨立的虛擬環境，但它們往往需要互相溝通。例如，一個 Web 應用容器可能需要連線到資料庫容器來存取資料。Docker 提供了多種方式來處理這件事，包括 Port Mapping 和 Linking。Port Mapping 是將容器內的端口暴露到宿主機上，讓外部存取；但 Linking 則是另一種更安全的內部互動方式，它在容器間建立隧道，讓接收端容器能看到來源端容器的特定資訊，而不用暴露到外部網路

## 容器之間的連線 (Linking)

在早期的 Docker 版本中，若要讓兩個容器互相溝通，官方提供的方式是使用 --link 參數建立「容器連線」

當運行 `docker run` 時，系統會自動分配一個隨機名稱（如 ospwn3y803hc），但這不好記。自訂名稱有兩個好處，一是更容易記住，例如將 Web 應用容器命名為 web、再來是作為連線其他容器的參考點，例如連線 web 到 db

```bash
sudo docker run -d -P --name web practice/webapp python app.py
```

```bash
# 檢查命名
sudo docker ps -l
CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
ospwn3y803hc  practice/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web
```

```bash
# 或用 docker inspect
sudo docker inspect -f "{{ .Name }}" ospwn3y803hc
/web
```

再來就是要連接了，使用 `--link`

```bash
sudo docker run -d --name db practice/postgres
```

```bash
# 再建立一個 web 容器，並讓它連線到 db
sudo docker run -d -P --name web --link db:db practice/webapp python app.py
```

```bash
# 檢查連線
docker ps
CONTAINER ID  IMAGE                     COMMAND               CREATED             STATUS             PORTS                    NAMES
349169744e49  training/postgres:latest  su postgres -c '/usr  About a minute ago  Up About a minute  5432/tcp                 db, web/db
aed84ee21bde  training/webapp:latest    python app.py         16 hours ago        Up 2 minutes       0.0.0.0:49154->5000/tcp  web
```

這裏可以看到 db 的 NAMES 欄有 web/db，表示連線成功。Linking 建立安全隧道，不需映射端口到宿主機，避免暴露資料庫到外部

## Docker Network drivers

Docker 的網路系統是 pluggable 的，也就是說它透過不同的 Network Drivers 來提供多樣化的網路模式與功能

### Bridge 預設網路驅動

如果在建立容器時沒有特別指定網路，Docker 就會使用 Bridge 模式。這是最常見的網路模式，適合容器之間在同一台主機上互相通訊的場景，容器有自己的私有 IP，對外連線透過宿主 NAT，通常是 Web 應用 + 資料庫在同一台主機上

### Host 共享宿主的網路堆疊

使用 Host 模式時，容器與宿主主機共用相同的網路環境，容器不再擁有獨立的 IP，而是直接使用宿主的網卡與 port，優點是效能佳、延遲低，但缺點是無網路隔離，可能 port 衝突，通常應用在需要監控代理、metrics exporter

### Overlay 多主機通訊的基礎

Overlay 網路用於連接多個 Docker Daemon 讓容器或 Swarm 服務能夠跨節點通訊，而不需設定 OS 層級的路由，主要使用在 Swarm、k8s，優點是簡化多節點部署，場景是建立微服務集群時

### IPvlan 控制 IP 與 VLAN 的進階方案

IPvlan 提供使用者對 IPv4/IPv6 網址的完整控制，VLAN 模式則可進一步實現 Layer 2 VLAN 標記與 L3 路由，適合需要與底層實體網路深度整合的場景，場景是在大型企業網路、資料中心

### Macvlan 讓容器看起來像實體機器

Macvlan 允許你給每個容器分配獨立的 MAC 位址，讓容器在網路上看起來就像獨立的實體主機，Docker 會根據 MAC 位址來路由封包，主要應用在遷移 legacy app 或需要實體化網路識別的應用

### None 完全隔離的網路模式

none 模式會將容器完全隔離，不與宿主或其他容器連線，此模式不適用於 Swarm 服務，主要用途是在建立無網路的安全或測試環境，場景是封閉式運算、離線任務、網路安全測試

### Third-party network plugins

Docker 也支援安裝第三方網路插件，讓使用者可整合特殊的網路堆疊或 SDN（Software-Defined Networking）方案，例如 Calico、Cilium 等

## `docker network ls`

我們可以利用以上的指令來查看目前的網路類型

```bash
docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
8ddb7e9846c6   bridge    bridge    local
48e785b7efb3   host      host      local
7e07c5b5ae34   none      null      local
```

## Bridge mode

我們比較常見的 docker 網路類型是 bridge mode

```bash
docker network create my-net

# 創建容器時指定 --network
docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest

docker network connect my-net my-nginx
```

```bash
# 創建網路時用 --ipv6 啟用 IPv6，若無指定 --subnet，會自動選擇 Unique Local Address (ULA) 前綴

docker network create --ipv6 --subnet 2001:db8:1234::/64 my-net

docker network create --ipv6 --ipv4=false v6net
```

## Reference

https://docs.docker.com/engine/network/drivers/
