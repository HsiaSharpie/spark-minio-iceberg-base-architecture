# spark-minio-iceberg-trino-base-architecture

提供了一個整合 Apache Spark、MinIO 和 Apache Iceberg 的參考架構與設置，相較於使用 Iceberg Rest 作為 Catalog，替換了以 Hive metastore 管理 catalog，metastore 是以 Postgres 架構，更換為 Hive metastore 的主要想法來自於 Trino 可共用這個 metastore，以支援即時查詢分析。

### 主要架構
1. **Apache Spark**：用於大規模資料的批量處理與進階分析。
2. **MinIO**：作為對象存儲系統，支援 S3 API。
3. **Apache Iceberg**：管理大規模資料集的表格格式。
4. **Hive Metastore**：作為 Iceberg 的元數據存儲，並與 Trino 共用以進行即時 SQL 查詢。
5. **Trino 整合**：透過共享的 Hive Metastore 提供即時的查詢與分析能力。

## 系統架構

系統包含以下元件：

1. **MinIO**：
   - 作為資料檔案的對象存儲後端。
   - 支援 AWS S3 API，易於與 Iceberg 整合。

2. **Hive Metastore**：
   - 存儲 Iceberg 表的元數據。
   - 由 Spark 與 Trino 共用，確保一致性與即時存取元數據。
   - 使用版本: 4.0.0

3. **Spark**：
   - 配置為與使用 Hive Metastore 管理的 Iceberg 表一起運作。
   - 支援批量處理與資料轉換。
   - 架構為較為簡單的 1 Master，1 Worker 沒有額外配置多個 Worker。
   - 使用版本: 3.5.3

4. **Trino**：
   - 通過共享的 Hive Metastore 存取 Iceberg 表。
   - 提供快速的即席 SQL 查詢能力。
   - 架構也為較為簡單的 1 Coordinator，無 Worker。

## 前置條件

- 安裝 Docker 與 Docker Compose
- 已安裝 Java

## 快速開始

1. 複製此儲存庫：
   ```bash
   git clone https://github.com/your-username/spark-minio-iceberg-hive.git
   ```

2. 啟動 Docker 化環境：
   ```bash
   docker-compose up -d
   ```

3. 配置 Spark 使用 Iceberg 與 Hive Metastore：
   - 確保 `hive.metastore.uris` 指向 Hive Metastore 服務。
   - Spark 配置範例：
     ```properties
     spark.sql.catalog.demo=org.apache.iceberg.spark.SparkCatalog
     spark.sql.catalog.demo.type=hive
     spark.sql.catalog.demo.uri=thrift://hive-metastore:9083
     spark.sql.catalog.demo.warehouse=s3://warehouse/wh/
     ```

4. 透過 Trino 查詢 iceberg 表格：
   - 更新 Trino 配置中的 `hive.properties` 文件：
     ```properties
     connector.name=iceberg
     hive.metastore.uri=thrift://metastore:9083
     hive.s3.aws-access-key=admin
     hive.s3.aws-secret-key=password
     hive.s3.endpoint=http://minio:9000
     hive.s3.path-style-access=true
     ```

## 目錄結構

```plaintext
.
├── docker-compose.yml       
├── hive/
│   ├── core-site.xml
│   ├── hive-site.xml        
│   └── lib/               
├── spark/
├── trino/
│   ├── etc
│        ├──catalog
│           ├── my_iceberg.properties
│           ├── ....
│── .env
│── .gitignore
└── README.md
```

## 參考

此設置為參考 Databricks 的 [docker-spark-iceberg](https://github.com/databricks/docker-spark-iceberg)  Repository，主要差異為使用 Hive Metastore 作為元數據存儲，而非 REST 目錄，並額外增添 Trino 以提供即時搜尋。