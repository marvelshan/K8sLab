1. What is a state in Terraform?

   詢問 Terraform 中「狀態 (State)」的定義，管理 metadata，記錄資源的相依性。Terraform 根據狀態中的相依關係圖來決定建立或銷毀資源的順序，對於支援的資源，Terraform 可以透過狀態檔來讀取資源的當前屬性，而不必每次都透過 Provider 去遠端查詢所有資源的完整細節，這大大加快了 `terraform plan` 的速度，當團隊多人共同管理基礎設施時，狀態檔案必須被共享，否則每個人的 Terraform 都會對基礎設施有不同的認知。為了解決這個問題，通常會使用 遠端狀態儲存 (Remote State Backend)，例如將狀態檔案存放在 Amazon S3、Azure Storage 或 HashiCorp Terraform Cloud 上，並透過鎖定機制來防止多人同時修改造成的衝突

   A: State is a JSON file which keeps track of metadata and mappings between resources and remote objects.

   正確答案。Terraform 狀態預設儲存在一個名為 terraform.tfstate 的檔案中，其格式是 JSON。這個檔案可以被讀取（但強烈建議不要手動編輯），資源與遠端物件的映射 (Mappings)，這是狀態最核心的功能。Terraform 程式碼（例如 aws_instance.web）只是一個「預期狀態」的宣告。狀態檔案則記錄了「這個 aws_instance.web 資源對應到雲端平台上哪一個『實際的』虛擬機器實例（例如 i-1234567890abc）」。沒有這個映射，Terraform 就無法知道要更新或刪除哪一個遠端物件

   B: State is an attribute of a Terraform resource describing its current lifecycle; for example, 'destroyed'.

   錯誤。Terraform 資源確實有生命週期狀態（例如 planned，created，destroyed），但這不是問題所指的「State」，「State」指的是整個 Terraform 專案的全域狀態，而不是單一資源的屬性

   C: State is an immutable type of Terraform resource.

   Immutable（不可變）是 Terraform 資源在更新時的一種行為模式（例如，變更 ami 會導致先建立新機器再刪除舊的），但它不是狀態本身的定義。狀態本身是可變的 (Mutable)。每當你執行 terraform apply 對基礎設施進行變更後，狀態檔案都會被更新以反映最新的情況

   D: State is an extension to Terraform which allows external providers to be used.

   Provider 是 Terraform 的外掛，用來與不同的 API（如 AWS, Azure, Google Cloud）或服務進行互動。狀態與 Provider 是兩個完全不同的概念

2. What is the 'taint' operation used for in Terraform?

   Terraform 中 taint 指令的用途

   A: It marks resources to be destroyed and recreated on the next execution of apply.

   正確答案。核心功能是 terraform taint 指令的設計目的，就是手動將一個資源標記為「已汙染」。當一個資源被標記為 tainted 後，Terraform 會在下次執行 terraform apply 時，先銷毀這個被汙染的資源，然後再建立一個新的來替代它。執行 taint 本身不會立即變更任何基礎設施，它只會修改狀態檔案，在該資源的記錄中加上汙染標記。真正的銷毀與重建發生在接下來的 apply 過程中

   - 使用時機：

     資源故障：你懷疑某個資源（例如一台虛擬機器）處於一個不良或損毀的狀態，手動修復很麻煩，希望透過重建來恢復。

     人為干預：有人在 Terraform 管理的資源外部進行了變更，導致資源狀態與 Terraform 的預期狀態偏離，而你希望讓 Terraform 重新接管，使其恢復到程式碼定義的狀態。

     安全理由：例如，你懷疑一台機器私鑰可能已經外洩，需要強制重建一台新的。

   > [reference](https://www.purestorage.com/tw/knowledge/what-is-terraform-taint.html)

   B: It allows properties of the resource to be changed directly in the Terraform state without using the apply command.

   錯誤。這個描述是在定義 `terraform state` 指令（特別是 terraform state rm、terraform state mv）的功能。這些指令允許你直接對狀態檔案進行操作，而不是 taint。taint 的結果是觸發「銷毀後重建」，而不是直接修改資源的某個屬性

   C: It rewrites all Terraform configuration files to a canonical format.

   這個描述是在定義 terraform fmt 指令。`terraform fmt` 會將你的 `.tf` 配置檔案重新排版成 Terraform 標準的格式風格，以確保一致性和可讀性

   D: It is used to convert HCL configuration files into JSON.

   錯誤。Terraform 支援兩種配置語法：HCL (.tf) 和 JSON (.tf.json)。雖然它們可以等價互換，但並沒有專門的 taint 指令來進行轉換。這個轉換通常需要手動或透過工具（如 `hcl2json`）來完成

   ### 補充

   1. taint 的替代方案：在較新版本的 Terraform 中，taint 指令雖然仍然有效，但官方更推薦使用 -replace 選項。

      指令比較：

      舊方式：terraform taint aws_instance.example

      新方式：terraform apply -replace="aws_instance.example"

      優點：-replace 選項能更好地與整個計畫流程整合，你可以在執行 apply 之前先透過 plan 來查看替換操作會帶來哪些具體影響。

   2. 與 destroy 的區別：

      terraform destroy：銷毀整個 Terraform 管理的基礎設施堆疊。

      terraform taint / apply -replace：僅針對單一資源進行銷毀與重建，堆疊中的其他資源保持不變。

3. When is State Locking performed in Terraform?

   什麼是狀態鎖定? 狀態鎖定是一種保護機制，用於防止多人同時對同一個狀態檔案進行寫入操作，從而避免狀態檔案損壞或出現競爭條件

   A: Locking happens automatically on any request to edit or read the state, and it blocks 'write' operations only.

   錯誤。狀態鎖定不會在「讀取」狀態時發生。執行 terraform plan 或 terraform show 這類僅讀取狀態的操作時，通常不會獲取鎖

   B: Locking happens automatically on any request to edit the state file, and it blocks all other I/O operations.

   錯誤。它不會阻塞所有其他 I/O 操作。鎖定主要阻塞的是其他「寫入」操作。多個「讀取」操作（例如多人同時執行 plan）通常可以並行進行

   C: Locking happens automatically on any request to edit or read the state file, and it blocks all other I/O operations.

   D: Locking happens automatically on any request to edit the state file, and it blocks 'write' operations only.

   正確。觸發時機：鎖定會在任何會「修改」狀態檔案的操作開始時自動進行，包括：

   terraform apply (創建、更新或銷毀資源)

   terraform destroy

   terraform refresh (舊版，現已整合進 plan 和 apply)

   terraform taint 和 terraform untaint

   terraform import

   執行帶有 -refresh-only 或 -replace 標誌的 apply 命令

   ### 補充

   - 鎖定是如何實現的？

     當使用本地狀態檔案（terraform.tfstate）時，Terraform 會使用作業系統的檔案鎖定機制。

     在團隊協作環境中，強烈推薦使用遠端狀態後端，例如 Amazon S3（配合 DynamoDB 表）、Azure Storage、Google Cloud Storage 或 Terraform Cloud/Enterprise。這些後端提供了更可靠、跨平台的鎖定機制（例如，在 S3 的物件上建立鎖定檔案，並在 DynamoDB 中記錄鎖定資訊）。

   - 鎖定流程：

     開始：在一個會修改狀態的操作（如 apply）開始時，Terraform 會嘗試從後端獲取一個鎖。

     執行：如果成功獲取鎖，則執行操作，並在執行期間一直持有該鎖。

     結束：操作完成（無論成功與否）後，Terraform 會釋放鎖。

     被阻塞：如果無法獲取鎖（因為被其他操作持有），Terraform 會顯示錯誤並中止，而不是強行執行。

   - 強制解鎖：

     如果一個操作異常終止導致鎖未被釋放（例如，網路中斷、程序被強制殺死），會造成「僵局鎖」。在這種情況下，管理員可以使用 terraform force-unlock 命令來手動釋放鎖。這個操作非常危險，必須在絕對確定沒有其他操作正在進行的情況下才能使用。

4. Select one false statement about Terraform imports:

   A: Importing may result in a "complex import" where multiple resources are imported at once and extra changes in the Terraform code are required to prevent their destruction.

   當導入一個資源時，Terraform 只會導入你明確指定的那個資源。如果這個資源在遠端系統中依賴於其他資源，或者你的 Terraform 程式碼結構不正確，可能會導致後續執行 `terraform plan` 時，Terraform 建議銷毀並重新建立某些資源。為了避免這種情況，你通常需要額外調整 Terraform 配置，使其與遠端基礎設施的精確結構相匹配，這可能涉及創建多個資源區塊或使用更複雜的資料結構

   > [reference](https://www.purestorage.com/tw/knowledge/what-is-terraform-import.html)

   B: Importing a resource does not require a state to be locked.

   錯誤。terraform import 是一個會修改狀態檔案的操作。它會將遠端資源的 ID 與 Terraform 狀態檔案中的資源地址建立映射關係。由於狀態鎖定的目的就是防止多人同時寫入狀態檔案，因此在執行 import 時，Terraform 會自動獲取狀態鎖。如果狀態已被其他操作鎖定（例如另一個團隊成員正在執行 apply），你的導入命令將會失敗

   C: Only some types of resources can be imported, and this depends on the specific implementation in the provider.

   一個資源類型能否被導入，完全取決於對應的 Terraform Provider 是否為該資源實現了 Importer 接口。雖然大多數常見的資源類型都支援導入，但並非全部。你需要在對應 Provider 的文件中查閱特定資源，以確認其是否支援導入功能

   D: It is possible to import resources with the "count" or "for_each" attribute defined in the configuration file.

   可以導入資源到使用了 count 或 for_each 的資源區塊中。這需要你在導入時使用正確的資源地址。對於 count：地址格式為 `<RESOURCE_TYPE>.<NAME>[<INDEX>]` (例如：`aws_instance.web[0]`)。對於 for_each：地址格式為 `<RESOURCE_TYPE>.<NAME>["<KEY>"]` (例如：`aws_subnet.private["us-east-1a"]`)

5. Select one false statement about Terraform's communication with providers:

   A: Terraform can use client pull network communication where the server provides the configuration from Terraform.

   這個描述試圖用「客戶端拉取」的網路通訊模型來描述 Terraform 與 Provider 的互動。Terraform Core 是「客戶端」，它啟動並「拉取」Provider 插件來執行工作。但說「伺服器提供來自 Terraform 的配置」這種說法並不準確。真正的通訊是程序間的通訊

   B: Communication between Terraform and its providers is JSON based.

   錯誤。Terraform Core 與 Provider 插件之間的通訊是透過 gRPC 協議進行的，而不是 JSON。gRPC 是一個高性能的遠端程序呼叫框架，使用 Protocol Buffers 作為接口定義語言和序列化機制

   C: The plugin used for communication with a specific provider must implement a golang abstraction layer which allows Terraform to communicate with its resources.

   Terraform Provider 確實是作為插件實現的，每個 Provider 都必須實現 Terraform 定義的特定 Go 接口。這些接口構成了抽象層，讓 Terraform Core 能夠以標準化的方式與所有不同的 Provider 進行互動，而無需了解每個雲端服務商的具體 API 細節

   D: Terraform can use server push network communication where Terraform pushes the configuration to the provider.

   Terraform Core 確實是主動的一方，它將規劃/配置資訊「推送」給 Provider 插件，然後 Provider 負責執行具體的資源操作（如調用雲端 API）。這種描述比選項 A 的「客戶端拉取」更符合實際的運作方式

   > [reference](https://cloud.tencent.com/developer/article/2416823)

6. Choose one sentence which does not compare data source and resource correctly:

   找出不正確比較資料來源與資源的敘述

   A: Both resources and data sources can use count/for_each to be created multiple times.

   正確的比較。可以使用 count 和 for_each 建立多個相同類型的資源實例，同樣可以使用 count 和 for_each 查詢多個資料實例。例如，根據列表中的多個 ID 查詢多個 AMI

   B: In order to access any of the data source objects, both data source and resource are to be accessed from a namespace called 'data' while resources may be reached directly via their types and names.

   錯誤的比較。這個敘述的前後矛盾。它說「為了訪問任何資料來源物件，兩者（資料來源和資源） 都要從稱為 'data' 的命名空間中訪問」，但這是不正確的。通過其類型和名稱直接訪問，格式為 <RESOURCE_TYPE>.<NAME>（例如：aws_instance.web），必須通過 data 命名空間訪問，格式為 data.<DATA_SOURCE_TYPE>.<NAME>（例如：data.aws_ami.ubuntu）。原句將資源也歸類為需要通過 data 命名空間訪問，這是錯誤的

   C: The data source is an immutable object, whereas resources may be modified.

   正確的比較。確實是不可變的。它們代表的是在特定時間點讀取的資料快照。執行 terraform apply 不會改變資料來源本身，只會重新讀取資料。如果遠端資料發生變化，Terraform 會偵測到這種變化並在下次執行時更新狀態，但這不是「修改」資料來源，而是用新資料替換舊快照。資源：是可管理的，可以透過 Terraform 進行創建、更新和刪除。它們的生命週期由 Terraform 管理

   D: Both resources and data sources can be imported from the Terraform CLI using import.

   當你有現有的基礎設施資源（例如已經在 AWS 上運行的 EC2），你可以將它們導入到 Terraform 的狀態文件中，這會將現有的 AWS EC2 實例導入到你的 Terraform 狀態中，對應到配置文件中的 aws_instance.example 資源

   ```bash
   terraform import aws_instance.example i-1234567890abcdef0
   ```

   Data sources 也可以被導入，雖然這較少使用。從 Terraform 1.5 版本開始，data sources 也支持 import block

   ```hcl
   import {
      to = data.aws_ami.example
      id = "ami-12345678"
   }
   ```

   ```bash
   terraform import data.aws_ami.example ami-12345678
   ```

   ### 補充

   - resource

     代表 Terraform 可以「建立或修改」的實體（例如 EC2 instance、S3 bucket）。會實際影響基礎設施狀態。Resources 直接通過類型和名稱訪問，不需要任何命名空間

     語法：

     ```hcl
      # 定義 resource
      resource "aws_instance" "web" {
         ami           = "ami-12345678"
         instance_type = "t2.micro"
      }

      # 訪問 resource - 直接使用類型和名稱
      resource "aws_eip" "ip" {
         instance = aws_instance.web.id  # 類型.名稱.屬性（沒有 data 前綴）
      }
     ```

   - data source

     是唯讀的查詢介面，用來從外部系統或現有資源中「讀取資料」。不會修改任何東西。Data sources 必須使用 data 命名空間前綴

     語法：

     ```hcl
      # 定義 data source
      data "aws_ami" "example" {
         most_recent = true
         owners      = ["amazon"]
      }

      # 訪問 data source - 必須加 data 前綴
      resource "aws_instance" "web" {
         ami = data.aws_ami.example.id  # data.類型.名稱.屬性
      }
     ```

7. Select a false statement about configuring providers in Terraform:

   選擇一個關於在 Terraform 中設定 provider 的錯誤陳述

   A: Each provider type can have only one configuration.

   每種 provider 類型只能有一個設定是錯誤的，實際上 Terraform 允許對同一個 provider 類型設定多個不同的 alias（別名）配置

   ```hcl
   provider "aws" {
      region = "us-east-1"
   }

   provider "aws" {
      alias  = "west"
      region = "us-west-2"
   }
   ```

   B: A resource can explicitly define which provider will manage its configuration.

   當你有多個同類型 provider 配置時，可以在 resource 中指定 provider 參數

   ```hcl
   resource "aws_instance" "example" {
      provider = aws.west
      ...
   }
   ```

   C: Providers can use attributes of other resources for configuration.

   Provider 設定是可以使用其他資源的屬性值，只要依賴順序明確且無循環，雖然在實務上這種依賴可能導致初始化問題（因為 provider 要先存在才能建 resource），但從語法上 Terraform 支援 provider 配置引用輸入變數或外部值

   ```hcl
   resource "aws_ssm_parameter" "region" {
      name  = "region"
      value = "us-west-2"
   }

   provider "aws" {
      region = aws_ssm_parameter.region.value
   }
   ```

   D: Provider configuration can be defined with variables or local values.

   Provider 設定可以透過變數或 local 值定義

   ```hcl
   variable "region" {
      default = "us-west-2"
   }

   locals {
      profile = "default"
   }

   provider "aws" {
      region  = var.region
      profile = local.profile
   }
   ```

8. What is a target in Terraform?
   Terraform 中「target」這個名詞的實際用途

   ### 補充

   Terraform 中的 “target” 是什麼？ Terraform 裡的「target」並不是一種語法關鍵字，而是 CLI 指令中的一個選項 `-target=ADDRESS` 用來指定「只對某個資源（或模組）執行 plan / apply / destroy」

   ```bash
   terraform apply -target=aws_instance.web
   ```

   A: A target is a keyword for a module configuration which defines its behaviour.

   錯誤。Terraform 裡「target」不是 module 的設定關鍵字，Module 沒有 target 這個參數，module 行為是透過 source、inputs 等參數控制的

   B: A target is a resource on which 'plan', 'apply' or 'destroy' commands will be executed.

   只針對被指定的資源或模組執行 plan / apply / destroy，這些指令只會作用在指定的「target」上

   ```bash
   terraform plan -target=aws_s3_bucket.logs
   terraform destroy -target=module.vpc
   ```

   C: There isn't anything called a target in Terraform.

   錯誤。有這個詞

   D: It is an interface for a provider whose API is being used by Terraform.

   錯誤。那是在講 provider interface，不是 target

9. Choose one false sentence about output in Terraform:

   ### 補充

   - Terraform output 是什麼?

     在 Terraform 中，output 用來 輸出某些值，方便在模組之間傳遞或顯示在 CLI。常見是將模組內的值傳出（module → root）將執行結果顯示在 CLI，讓外部工具（如 CI/CD pipeline）讀取

     ```bash
     output "instance_ip" {
        value = aws_instance.web.public_ip
     }
     ```

   A: Output values may be used to expose data outside of a Terraform module.

   正確。Output 讓模組之間能互通資料

   B: It is possible to prevent sensitive data of outputs from being displayed in the CLI.

   正確。Terraform 有 sensitive = true 參數可以設定 output，避免在 plan 或 apply 時將敏感資料顯示在終端機上

   ```
   output "db_password" {
      value     = aws_db_instance.main.password
      sensitive = true
   }
   ```

   C: Outputs are stored in the Terraform state.

   正確。所有 output 值都會存在 terraform.tfstate 檔裡，Terraform 需要從 state 中知道這些 output 的實際值

   D: Only string values may be used as output.

   output 支援任何型別，string、number、bool、list、map、object、甚至 nested 結構

   ```bash
   output "server_info" {
      value = {
         ip   = aws_instance.web.public_ip
         name = aws_instance.web.tags.Name
      }
   }
   ```

10. What type of provisioning is not supported by Terraform?

- Provisioners 是 Terraform 在建立或銷毀資源時，用來在機器上執行命令或設定檔案的工具，常見用途是建立完 EC2、VM、Container 後安裝軟體或複製設定檔、執行 shell script。內建 provisioners 設定包括 `local-exec` 在本地端執行命令、`remote-exec` 透過 SSH 或 WinRM 在遠端執行命令、`file` 將檔案上傳至遠端主機

  A: Executing commands on every planned application.

  Terraform 的 provisioner 只在資源建立或銷毀時執行，而不是「每次執行 plan 或 apply」都觸發，Provisioner 是「生命週期階段觸發」的（create/destroy）

  ```hcl
  provisioner "remote-exec" {
      inline = ["sudo apt update", "sudo apt install nginx -y"]
   }
  ```

  B: Creating files.

  有內建 file provisioner

  ```hcl
  provisioner "file" {
      source      = "app.conf"
      destination = "/etc/app.conf"
   }
  ```

  C: Defining external provisioners like puppet/chef.

  Terraform 支援外部 provisioners，例如：chef、puppet、ansible

  D: Executing commands over SSH.

  透過 remote-exec provisioner，Terraform 會自動連線執行

  ```hcl
  provisioner "remote-exec" {
   inline = ["sudo systemctl restart nginx"]
   connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = self.public_ip
      private_key = file("~/.ssh/id_rsa")
   }
   }
  ```
