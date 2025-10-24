## K8S Lab Day_39

## 前言

因為前幾天有發生 AWS 重大的事件也就是 region US-EAST-1 大停機，導致上千個應用程式當機無法處理 request，包括 Snapchat、Fortnite、Roblox、Coinbase 和 Canva 等等，持續了很長的一段時間，我注意到的時候下中午，我記得到當天傍晚我在看 medium 還是 500，我也在網路上爬了一些資料，今天就來討論一下這次級聯性的問題吧

## 事件概述與時間脈絡

在 2025/10/20 12:11 AM PDT(太平洋時區)也就是台灣時間下午 3:11，AWS 的 health check dashboard 出現 US-EAST-1 區域的多個服務出現錯誤率上升與延遲增加，高峰期，Downdetector 記錄超過 15,000 起用戶投訴，全球網站中斷報告快速增加，AWS 於約 3:53 PM PDT 宣布初步恢復，但部分服務如 EC2 新實例啟動仍需數小時清除積壓，這也不是網路攻擊，而是內部基礎設施問題，AWS 歸因於 DynamoDB API 端點的 DNS 解析故障，因為 US-EAST-1 是 AWS 的核心 region，承載全球控制平面像是 IAM 更新與 DynamoDB，許多跨區域服務依賴其穩定性，事件雖限於單一區域，卻因 AWS 服務間的緊密耦合，迅速擴散至全球

## 從 DNS 解析到身份驗證崩潰

假如要理解這次的故障需要先了解 AWS 的幾個知識點，第一是 DNS，也就是雲端服務的電話簿，負責將域名轉換成 IP address，像是 google.com 轉換成 8.8.8.8，而在 AWS 裡就是 dynamodb.us-east-1.amazonaws.com，轉換成內部節點的 IP address、再來是 DynamoDB，它作為 AWS 的 fully managed NoSQL 資料庫，專門是高可用的存在，用於儲存應用狀態和用戶資料等等的資料、再來是 API endpoint 是 Lambda、EC2 等服務存取 DynamoDB 的入口

故障的原因是在 US-EAST-1 的 DynamoDB API 端點 DNS 解析失敗，這個可能起因於內部的某個 bug 導致 health check 中斷，進而 blcok API request，看起來好像是小問題，但是 DynamoDB 是許多後端 server 依賴於他，像是 EC2 啟動 instance 需要查詢 DynamoDB 中的配置資料，DNS 故障導致服務沒辦法正確的解析 IP 位置，導致 request 暴增

再來是這個問題觸發了 IAM 的連鎖反應，IAM 是 AWS 身份和存取管理服務，負責定義 Roles 和 Policies，確保服務之間的安全溝通，AWS 不直接使用 IAM 發的憑證，而是透過 STS(Security Token Service)，取得 Temporary Security Credentials，這些憑證有效期限通常為數小時或到數天都有

STS 是 regionalized 像是美西與東京的 STS 獨立運作，而他用於頻繁的憑證更新，用來降低 IAM 的負載，IAM 本身是全球性的服務，透過 edge locations 全球部署，提供及快速的 policy 查詢，但是這次的事件中，IAM 更新 dependency 於 US-EAST-1 的 DynamoDB endpoint，而 DNS 故障導致 IAM 無法存取必要的資料，進而阻礙 STS 憑證的發放，沒有有效的憑證，服務間沒辦法互信，導致 Lambda 呼叫不到 DynamoDB，EC2 連不上 S3，甚至 EventBridge 事件無法觸發下游流程

所以這也導致了 single point of failure (SPOF)，儘管 IAM 和 STS 設計是高可用的，卻因為 DynamoDB 隱藏的 dependency 而產生風險

## Cascading Effect: Serverless 架構的放大鏡

AWS 的事件影響範圍相當的大，但不同的架構也有不同的差異性，而這次故障特別凸顯的是 serverless 的架構，而 Serverless 架構以 Lambda、S3 和 DynamoDB 為核心三件套，Lambda 依賴容器化執行環境，AWS 會根據流量動態分配資源，有使用過 Lambda 或是考過 AWS certification 的就知道他會有 cold start 的狀況，當閒置的容器回收時，新事件觸發會需要重新開始初始化，當然包括取得 STS 憑證並且查詢 IAM policy

在高流量的應用中， cold start 常常發生，像是 Canva 再上傳圖檔到 S3 的時候，會觸發 Event Notification 然後進而 cold start Lambda 處理圖像轉換，假如這時候 IAM 沒有正常的運作，cold start 會失敗，導致整個 workflow 中斷、Perplexity 等 AI 應用也依賴於 Lambda 查詢 DynamoDB，流量高峰時冷啟動需求疊加，放大故障影響

相對的 EC2 這種傳統的 instance，一但啟動並且取得憑證之後就可以獨立運作幾小時或是數天，不需要頻繁的查詢 IAM，這次事件中多數傳統的 stateful instance 都維持正常運作，反而成為系統穩定的關鍵，這讓人不禁反思我們過去是否在未充分理解底層機制的情況下，為了節省成本而盲目追求 serverless？還是說，選擇傳統架構、意外避開這次災情，才更接近所謂的「正確」選擇？

在網路監控中，在 serverless 服務中錯誤率高峰打 80%，而 EC2 延遲僅上升 20-30%，這也反映了 Serverless 追求彈性與成本效率，但犧牲部分預測性，EC2 則強調穩定，適合關鍵業務

## 全球影響與經濟代價

這次的事件波及到了社交媒體、遊戲、金融和平常會用到的生產力工具，估計的經濟損失相當的大，凸顯了雲端廠商的 vendor lock-in 風險，怎麼來去思考服務的穩定性也是接下來我們更要去思考的問題

## 教訓與未來展望

這是的事件也讓我們更需要在建立架構時考慮更多的狀況，也許是採用 Multi-Region、異地備援 STS 憑證輪換，以及監測 DNS 與健康檢查、非關鍵部分再導入 Serverles 等等，AWS 已承諾發布完整事後報告，預計將強化 DynamoDB 端點的 DNS 冗餘

## History (The past) does not repeat itself, but it (often) rhymes\_\_Mark Twain

歷史的細節可能不同，但總會出現驚人的相似之處或模式，來自馬克吐溫，回顧 2021 年 12 月 7 日與 10 日的 AWS US-EAST-1 中斷事件，雖然那時候我還沒有踏入業界，這次的事件也讓我去回顧了過去的歷史事件，因為資料中心硬體故障，導致 internal network block，而這次出事的也是這個 region，這也是 AWS 最早的 region 也是大多數服務的 region，然後也是最便宜的，這也影響了相當多的服務，在之後 AWS 強化了區域隔離與自動 failover 機制，強調多 Multi-AZ 的重要性

而時間來到 2025/6/12，GCP 也迎來了一場更大的全球性中斷，這次事件從美國太平洋時間上午 10:49 開始，歷時八小時，造成超過 50 項服務停擺、影響 140 萬筆用戶。從 Gmail、Drive、YouTube 到 Spotify、Discord、Cloudflare，幾乎整個雲端生態都受到波及，這起事故並非來自硬體或網路，而是一次簡單的程式邏輯錯誤——一個漏掉空值檢查的升級版本，意外讓全球 42 個分區的 Service Control 系統同時因 Null Pointer Exception 停擺，修補雖在 40 分鐘內完成，但後續因重試機制缺乏隨機延遲與指數退避，造成雪崩式的重試流量，使部分大型分區如 US Central 1 進一步崩潰，最後運維團隊只能手動限流、重新分配流量，歷經三個多小時才逐步恢復。而這次事故也連帶拖垮了高度依賴 GCP 的 Cloudflare，讓全球 20% 的網路流量受到影響

接著要講到 2024/7/19 的事件，當天凌晨 4 點 9 分，全球數以百萬計的 Windows 系統幾乎同時陷入 Blue Screen of Death，從航空公司到電視台、銀行到醫院無一倖免，罪魁禍首是 CrowdStrike 為其 Falcon 安全防護軟體發佈的一次更新，這套軟體運作在作業系統的核心層，用於偵測惡意行為與防範威脅，但這次更新的「Channel File 291」出現邏輯錯誤，導致 Falcon 驅動程式 csagent.sys 在載入時觸發 `PAGE_FAULT_IN_NONPAGED_AREA` 錯誤，使電腦進入藍屏循環或無限重開機狀態，這場事故雖然不像雲端中斷那樣牽涉到分散式系統或資料中心，但波及範圍更為廣泛，任何安裝了 CrowdStrike Falcon 的 Windows 10、Windows 11 系統都可能中槍，大量企業伺服器、機場登機系統、甚至媒體播控機房全數停擺，全球運輸、金融與媒體網路一度陷入混亂

## 奇犽與小結

這幾起事件雖然都是發生在不同的層面，卻反覆地揭露了相同的教訓，即使是最頂尖的團隊、最成熟的系統，也可能因一個小小的錯誤、未加檢查的空值、或一次更新的疏忽而引發全球性災難

從程式邏輯錯誤到安全帶裡的更新事件，這些是見不得讓人重新思考可靠性的含義，可靠性不只是系統穩定的結果，更是一種對失誤的承認與設計，承認錯誤無可避免，並在架構中預留緩衝、隔離與回復的機制

每一次事故，都是整個產業的集體學習，我們要從這些事件中不斷吸取經驗、修正假設、精進設計，讓可預期的失誤變得更可控，也讓下一次災難發生時，我們能更快、更穩地站起來！

小備注：我也會持續的寫文章，放在我的 [github](https://github.com/marvelshan/tech-forum) 呦

## Reference

[Amazon says systems are back online after global internet outage](https://edition.cnn.com/business/live-news/amazon-tech-outage-10-20-25-intl)

[AWS Service health](https://health.aws.amazon.com/health/status?eventID=arn:aws:health:us-east-1::event/MULTIPLE_SERVICES/AWS_MULTIPLE_SERVICES_OPERATIONAL_ISSUE/AWS_MULTIPLE_SERVICES_OPERATIONAL_ISSUE_BA540_514A652BE1A)

[DynamoDB down us-east-1 by Reddit](https://www.reddit.com/r/aws/comments/1obd3lx/dynamodb_down_useast1/)

[AWS 大當機事件的真正問題](https://www.linkedin.com/posts/leehappy_aws-%E5%A4%A7%E7%95%B6%E6%A9%9F%E4%BA%8B%E4%BB%B6%E7%9A%84%E7%9C%9F%E6%AD%A3%E5%95%8F%E9%A1%8C-activity-7386565401750642689-J-6H)

[前幾天 AWS 的 us-east-1 出事報告](https://blog.gslin.org/archives/2021/12/14/10461/%E5%89%8D%E5%B9%BE%E5%A4%A9-aws-%E7%9A%84-us-east-1-%E5%87%BA%E4%BA%8B%E5%A0%B1%E5%91%8A/)

[Summary of the AWS Service Event in the Northern Virginia (US-EAST-1) Region](https://aws.amazon.com/tw/message/12721/)

[神仙打鼓有時錯 - DevOps 模範生 Google Cloud 6/12 全球大當機事件](https://blog.darkthread.net/blog/google-612-outage/)

[CrowdStrike 全球當機事件：資安長的反思與應對](https://www.cio.com.tw/crowdstrike-global-crash-event-the-ministers-reflections-and-response/)

[2024 年 CrowdStrike 大規模藍白畫面事件](https://zh.wikipedia.org/zh-tw/2024%E5%B9%B4CrowdStrike%E5%A4%A7%E8%A7%84%E6%A8%A1%E8%93%9D%E5%B1%8F%E4%BA%8B%E4%BB%B6)
