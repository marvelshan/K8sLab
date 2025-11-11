1. Select a Dockerfile command required to execute a build:

   選擇一個 build 時必需的 Dockerfile 指令

   A: MAINTAINER

   用來指定維護者資訊的指令（現已廢棄，改用 `LABEL maintainer=`）

   B: FROM

   如果 Dockerfile 中沒有 FROM，docker build 會失敗（除了極少數的 scratch 映像檔情況，但即使如此，FROM scratch 也是明確寫出的）

   C: ARG

   用來定義建置時的變數

   D: FROM but only when using Docker multistage

   multistage 只是多個 FROM 的使用方式，但任何 Dockerfile 都必須至少有一個 FROM，不僅限於多階段建置

   > [reference](https://tachingchen.com/tw/blog/docker-multi-stage-builds/)

2. The command for listing all containers in Docker is:

   列出 Docker 中所有容器的命令是什麼？

   A: docker container ls -a

   正確。`docker container ls` 列出運行中的容器，加上 `-a` 或 `--all` 會顯示所有容器（包括已停止的）

   B: docker ls --all

   不是有效的 Docker 命令

   C: docker show --all

   不是有效的 Docker 命令

   D: docker ps -aux

   docker ps 是傳統的列出容器命令，但 -aux 不是正確參數，docker ps -a 才顯示所有容器，-aux 是 Linux ps 命令的參數

3. How to connect from one container to another within the same Docker network?

   在同一個 Docker 網路中，如何從一個容器連接到另一個容器？當容器在同一個自定義 Docker 網路中時，Docker 會為每個容器提供內建 DNS 解析，可以使用容器名稱直接作為主機名來訪問，並支援 TCP 和 UDP 通訊

   A: You can use Docker API to ask the hosting system for IPs of the other containers in order to reach them via both UDP and TCP

   不正確。雖然可以透過 Docker API 查詢 IP，但這不是標準做法，而且太複雜。在同一個 Docker 網路中，不需要查詢 IP，可以直接用容器名稱訪問

   B: You can use container's name to reach it via TCP

   只能訪問 TCP 服務的說法太局限

   C: You can use container's name to reach it via UDP

   不完整。只能訪問 UDP 服務的說法太局限

   D: You can use container's name to reach it via TCP or UDP

   正確。容器名稱可以解析為 IP，然後透過 TCP 或 UDP 協議訪問（取決於目標容器監聽的協議）

   ## 補充

   這個功能只在自定義 Docker 網路中有效，預設的 bridge 網路需要額外設定，容器名稱由 Docker 的內建 DNS 自動解析，支援所有 IP 基礎的協議

   > [reference](https://philipzheng.gitbook.io/docker_practice/network/linking)

   > [docker 進階網路設定](https://philipzheng.gitbook.io/docker_practice/advanced_network)

   > [Docker Bridge Network](https://godleon.github.io/blog/Docker/docker-network-bridge/)

4. The command for transferring files from a host to a Docker container is:

   將檔案從 host 傳輸到 Docker container 的命令是什麼？

   A: docker cp foo.txt mycontainer:/foo.txt

   正確。docker cp 是 Docker 官方提供的檔案複製命令，用於在主機和容器之間複製檔案或目錄。`docker cp <主機路徑> <容器名稱>:<容器路徑>`

   B: docker mv foo.txt mycontainer:/foo.txt

   不是有效的 Docker 命令

   C: rsync foo.txt mycontainer:/foo.txt

   rsync 是 Linux 的遠端同步工具，但預設情況下它無法直接識別 mycontainer 這樣的 Docker 容器名稱，除非容器正在運行且配置了 SSH，否則無法直接使用

   D: docker sync -R foo.txt mycontainer:/foo.txt

   不是有效的 Docker 命令

   ### 補充

   從主機複製到容器：docker cp /host/path container:/container/path

   從容器複製到主機：docker cp container:/container/path /host/path

5. What is the purpose of using EXPOSE command in Dockerfile?

   在 Dockerfile 中使用 EXPOSE 指令的目的是什麼？在 Dockerfile 中，EXPOSE 用於聲明容器運行時將監聽的端口，要讓外部訪問，仍然需要在 docker run 時使用 -p 參數來 port mapping

   A: Exposing an image on another machine

   錯誤，EXPOSE 與將映像檔發佈到另一台機器無關

   B: Exposing volumes from one container to another

   錯誤，volumes 的共享是透過 -v 或 --mount 參數，與 EXPOSE 無關

   C: Exposing a specified port on which application is listening in the runtime

   正確。EXPOSE 是聲明容器運行時監聽的 port

   D: Exposing an image for specified user groups

   與用戶權限無關

6. What is a Docker hub?
   Docker Hub 是 Docker, Inc. 提供的雲端儲存庫服務（Registry Service），功能為：儲存與分發 Docker 映像檔（Docker images），提供公開映像檔（如官方 nginx、mysql、ubuntu 等），允許用戶上傳、下載、管理自己的映像檔，支援自動化建置（Automated Builds）與 Webhook

   A: It's a service for connecting multiple containers before deployment

   錯誤，容器連接是透過 Docker 網路或 Docker Compose 實現，與 Docker Hub 無關

   B: It's a registry service which allows users to download, share and manage Docker images

   正確，這正是 Docker Hub 的核心功能——一個集中式的 Docker 映像檔註冊服務

   C: It's a tool for managing volumes among containers

   volumes 管理是 Docker engine 本身的功能，不是 Docker Hub 的用途

   D: It's a registry of all built and deployed images

   不精確，Docker Hub 確實是一個 registry，但「所有已建置和部署的映像檔」這個說法太絕對，因為還有其他 registry（如 GitHub Container Registry、Google Container Registry 等），且不是所有映像檔都會上傳到 Docker Hub

7. How to start a terminal session of an already running container?

   如何啟動一個已經在運行的容器的終端機 session？

   A: It's impossible to enter a running container if it wasn't started in the interactive mode (-it).

   錯誤。即使容器啟動時沒有 `-it`，只要容器內有 `/bin/bash` 或類似 shell，仍然可以用 `docker exec -it` 進入。`-it` 是在 exec 時附加的，不是必須在 run 時就設定

   B: docker container exec <container_id> /bin/bash

   缺少 `-it`（interactive + TTY）參數，這樣執行會無法進行互動式操作。語法不完整，會進入 bash 但無法正常輸入命令

   C: docker run -it <container_id> /bin/bash

   docker run 是用來創建並啟動新容器，不是進入已運行的容器。這會嘗試基於指定容器 ID 的映像檔啟動新容器，而不是進入現有容器

   D: docker exec -it <container_id> /bin/bash

   正確！docker exec 是在運行中的容器內執行額外命令。`-it` 提供互動式 TTY 會話。`<container_id>` 指定目標容器。`/bin/bash` 是要執行的 shell

   ### 補充

   實際操作時，常用：

   ```bash
   docker exec -it <container_id> /bin/bash
   ```

   或

   ```bash
   docker exec -it <container_id> /bin/sh
   ```

   取決於容器內可用的 shell。

   **`-it` 實際上是：**

   `-i` 或 `--interactive`：保持 STDIN 開啟（標準輸入），沒有 -i 的話，你輸入的任何東西都不會傳送到容器內

   `-t` 或 `--tty`：分配一個偽終端（pseudo-TTY），模擬真實的終端環境，提供更自然的終端體驗：支援命令歷史、游標移動、Ctrl+C 中斷等

   - 只有 -i（沒有 -t）

     ```bash
     docker exec -i container_id /bin/bash
     ```

     可以輸入命令，但終端體驗很差，沒有提示符、不能使用方向鍵、Tab 自動補全失效，感覺像是在用原始的輸入管道

   - 只有 -t（沒有 -i）

     ```bash
     docker exec -t container_id /bin/bash
     ```

     有終端介面，但無法輸入命令，看起來正常，但當你打字時沒有任何反應

8. How to disable a specific container? Choose one wrong answer:

   如何停用一個特定的容器？選擇一個錯誤的答案。「停用」通常指讓容器停止運行，但不刪除它

   A: docker stop <container_id>

   正常停止容器，發送 SIGTERM 信號，允許容器優雅結束。容器會進入 Exited 狀態，但配置和檔案系統保留

   > [reference](https://ithelp.ithome.com.tw/articles/10304865)

   B: docker kill <container_id>

   強制立即停止容器（預設發送 SIGKILL）。容器也會進入 Exited 狀態，但屬於強制殺死，這也是「停用」的一種方式（雖然不優雅）

   C: docker rm <container_id>

   錯誤。刪除容器，不僅停止運行，還會移除容器的所有配置和儲存層（除非有 volume 保留資料）。這不是「停用」，而是「刪除」，容器將不復存在。這是錯誤答案，因為題目問的是「停用」不是「刪除」

   D: docker container pause <container_id>

   暫停容器的所有進程（使用 cgroups freezer），容器狀態變為 Paused。也是一種「停用」方式，可以稍後用 unpause 恢復

   > [reference](https://kernel.meizu.com/2024/07/12/sub-system-cgroup-freezer-in-Linux-kernel/)

9. What is the correct way to pass an argument with a value during Docker build phase?

   在 Docker build phase 傳遞帶有值的參數的正確方法是什麼？

   ### Docker 建置參數（--build-arg）機制

   在 Docker 建置過程中，可以使用 ARG 指令定義建置時變數：

   Dockerfile 中：

   ```dockerfile
   ARG MY_VAR
   RUN echo "Value is $MY_VAR"
   ```

   建置命令中：

   ```bash
   docker build --build-arg MY_VAR=my_value -t myimage .
   ```

   A: Adding --build-arg [KEY=VALUE,] flag to the docker build command.

   正確，這是 Docker 官方提供的在建置時傳遞參數的方法

   B: Docker does not use any build arguments.

   錯誤，Docker 確實支援建置參數（ARG）

   C: Setting it as an environment variable available on the machine executing a given build.

   雖然環境變數在某些情況下可能被讀取，但這不是 Docker 官方推薦或標準的建置參數傳遞方式。Docker 不會自動將主機環境變數作為建置參數傳入

   D: Defining it in a .env file in the root directory.

   .env 檔案用於 docker-compose，不是用於 docker build 的建置參數。docker build 不會自動讀取 .env 檔案中的變數

   ### 補充

   C 選項，官方推薦或標準的建置參數傳遞方式，主要是明確性與可重現性問題，任何人都可以複製貼上這個命令得到相同結果，假如是使用環境變數的方式，建置命令不完整，依賴外部狀態，其他人執行時可能忘記設定環境變數，導致建置失敗或產生不一致的 image

   > [reference](https://docs.docker.com/build/building/variables/?utm_source=chatgpt.com)

   > Build arguments and environment variables are inappropriate for passing secrets to your build, because they're exposed in the final image. Instead, use secret mounts or SSH mounts, which expose secrets to your builds securely.

   用 `--build-arg` 加上 ARG 指令可以清楚地表示「這是建置時變數」，並且只在建置期間有效，避免影響執行時的行為。官方文章說明：「If you need build-time customization, ARG is your best choice. If you need run-time customization … ENV is well‐suited.” ，使用 ENV 作為建置參數，會留下值在 image 中，讓後續運行者或其他人看到該參數，導致可預期性與版本控制較難維護

10. What is the difference between EXPOSE and PUBLISH commands?

- PUBLISH（實際是 -p 或 --publish 參數）是 docker run 的參數，例如 `docker run -p 8080:80`，主要作用是建立主機端口與容器端口的映射，讓外部可以訪問

  A: When not specified neither EXPOSE nor -p, the service in the container will only be accessible from outside of the container itself.

  錯誤。如果既沒有 EXPOSE 也沒有 -p，服務只能從容器內部訪問，同一網路下的其他容器也無法訪問（除非使用 --net=host 等特殊模式）

  B: If you PUBLISH a port, the service in the container is not accessible from outside Docker, but from within other Docker containers.

  錯誤。`-p` 發布端口的主要目的就是讓外部（主機及外部網路）可以訪問，不僅限於其他容器

  C: If you EXPOSE and -p <port>, the service in the container is accessible from anywhere, even outside Docker.

  正確。EXPOSE 聲明容器監聽的端口，`-p` 建立主機與容器的端口映射，以起使用後，服務可以從任何地方訪問（包括外部網路）

  D: When EXPOSE command is defined, ports can be accessible for the host.

  錯誤。單獨使用 EXPOSE 不會讓主機可以訪問容器端口，必須配合 -p 發布或使用 -P 自動映射
