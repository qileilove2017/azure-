Day 5：Azure Storage（Blob、File、Table）
模块	内容
主题	Azure Storage（Blob、File、Table）
学习目标	理解 Azure Storage 的结构与类型，掌握 Blob（对象存储）、File（文件共享）、Table（NoSQL存储）的使用场景与基本操作。
实战任务	创建 Storage Account，上传文件到 Blob 容器，配置访问策略，并挂载 Azure File 共享到本地系统。
🧠 一、核心知识讲解
1. 什么是 Azure Storage Account

Storage Account 是 Azure 存储服务的顶层容器，所有数据类型都在其中管理。

它支持四种主要类型：

存储类型	描述	典型场景
Blob Storage	用于存储非结构化对象数据（图片、视频、文档、备份）	网站静态资源、备份、AI训练数据
File Storage	提供 SMB 网络文件共享	应用日志共享、跨VM共享目录
Table Storage	NoSQL 键值存储	元数据管理、日志存储
Queue Storage	消息队列系统	解耦异步任务处理

💡 存储帐户命名规则：

全局唯一；

只能使用小写字母和数字；

长度为 3–24 个字符。

2. 存储层级结构
Storage Account
 ├── Blob Service
 │     └── Container（容器）
 │           └── Blob（文件对象）
 ├── File Share
 │     └── Directory / File
 ├── Table
 │     └── Entities（行记录）
 └── Queue
       └── Messages

3. 存储访问方式
方式	特点	示例
Portal	可视化操作	上传文件、设置访问级别
CLI / PowerShell	自动化管理	批量操作
SDK / REST API	应用集成	程序读取与写入数据
SAS（Shared Access Signature）	临时访问授权	可控时效、权限、路径
4. 存储层级（性能级别）
层级	描述	适用场景
Hot	高频访问	Web 应用文件、实时数据
Cool	低频访问	备份、日志
Archive	极低频访问	合规归档、冷数据
🧰 二、实战任务
任务 1：创建 Storage Account
az storage account create \
  --name mystorage$RANDOM \
  --resource-group learn-azure-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot


验证创建：

az storage account list --output table

任务 2：创建 Blob 容器并上传文件

获取连接字符串：

az storage account show-connection-string \
  --name mystorage123 \
  --resource-group learn-azure-rg \
  --query connectionString -o tsv


创建容器：

az storage container create \
  --name mycontainer \
  --account-name mystorage123 \
  --public-access blob


上传文件：

echo "Hello Azure Blob!" > test.txt

az storage blob upload \
  --account-name mystorage123 \
  --container-name mycontainer \
  --name test.txt \
  --file test.txt


验证上传：

az storage blob list \
  --account-name mystorage123 \
  --container-name mycontainer \
  --output table


💡 访问 Blob 文件：
在浏览器中访问：

https://mystorage123.blob.core.windows.net/mycontainer/test.txt


如果容器为公开访问（Public Access），即可直接打开。

任务 3：创建 Azure File 共享并挂载

创建文件共享：

az storage share create \
  --name myshare \
  --account-name mystorage123


上传文件到共享：

az storage file upload \
  --account-name mystorage123 \
  --share-name myshare \
  --source ./test.txt


获取连接字符串：

az storage account keys list \
  --account-name mystorage123 \
  --output table


在 macOS/Linux 上挂载 SMB 文件共享：

sudo mkdir /mnt/azfiles
sudo mount -t cifs //mystorage123.file.core.windows.net/myshare /mnt/azfiles \
  -o vers=3.0,username=mystorage123,password=<your-key>,dir_mode=0777,file_mode=0777


验证挂载：

ls /mnt/azfiles

任务 4：（进阶）创建 Table 存储并插入记录
az storage table create \
  --name MyTable \
  --account-name mystorage123

az storage entity insert \
  --account-name mystorage123 \
  --table-name MyTable \
  --entity PartitionKey=User RowKey=001 Name="ZhiHu" Role="Engineer"


查询表数据：

az storage entity query \
  --account-name mystorage123 \
  --table-name MyTable \
  --output table

任务 5：生成临时访问链接（SAS Token）

生成 SAS 链接（有效期 30 分钟）：

az storage blob generate-sas \
  --account-name mystorage123 \
  --container-name mycontainer \
  --name test.txt \
  --permissions r \
  --expiry $(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')


拼接访问：

https://mystorage123.blob.core.windows.net/mycontainer/test.txt?<SAS Token>

🧩 三、验证成果

✅ 你应能完成以下操作：

创建一个标准存储帐户；

上传并访问 Blob 文件；

挂载 Azure File 共享；

创建 Table 并插入数据；

使用 SAS 链接安全访问对象。

🧠 四、延伸与思考

安全性思考

生产环境应禁用“Public Access”，改用 SAS 或 Azure AD；

建议启用“防火墙 + 专用终结点 (Private Endpoint)”；

使用托管身份连接其他 Azure 服务（如 Function、App Service）。

性能优化

高频访问用 Hot 层；

冷数据使用 Cool/Archive；

对大量小文件，建议使用 Blob Batch 操作。

备份与灾备策略
Storage SKU（冗余模型）：

LRS：单区域冗余；

GRS：异地区域备份；

ZRS：区域可用区冗余；

RA-GRS：带只读副本的 GRS。

推荐学习资源

Azure Blob Storage 官方文档

Azure Files 官方文档

Azure Table Storage 官方文档

📘 今日总结

你了解了 Azure Storage 的类型、用途与架构；

掌握了 Blob 上传、File 共享挂载、Table 插入记录；

理解了访问控制（SAS、密钥、权限）；

建立了存储层自动化操作的基础；

具备使用 CLI 实现存储自动化的能力。