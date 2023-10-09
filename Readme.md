# starrocks-iceberg docker-compose

## 使用 Trino 写入 Iceberg 数据

先创建 schema 一定要指明 location
```sql
CREATE SCHEMA if not exists warehouse.silver
WITH (location = 's3a://warehouse/');
```

生成 Iceberg Table
```sql
CREATE TABLE warehouse.silver.orders
WITH (partitioning = ARRAY['month(orderdate)'])
AS
select * from tpch.tiny.orders;
```

```sql
CREATE TABLE warehouse.silver.customer
WITH (partitioning = ARRAY['bucket(custkey, 10)'])
AS
select * from tpch.tiny.customer ;
```
查询
```sql
select * from warehouse.silver.orders;

select count(1) from warehouse.silver.orders;
```

## 看 log
```bash
docker-compose logs  -f --tail 100
docker-compose logs -f --tail 100
docker-compose logs starrocks -f --tail 100
```

## Starrock 连接 Iceberg
```sql
CREATE EXTERNAL CATALOG warehouse
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://metastore:9083",
    "iceberg.catalog.warehouse" = "s3a://warehouse/",
	
	"aws.s3.enable_ssl" = "false",
	"aws.s3.enable_path_style_access" = "true",
	"aws.s3.endpoint" = "http://minio:9000",
	"aws.s3.access_key" = "minio",
	"aws.s3.secret_key" = "minio123"
)

SHOW CATALOGS;

SHOW DATABASES FROM warehouse;

SET CATALOG warehouse;

SHOW TABLE STATUS FROM warehouse.silver;

SELECT count(*) FROM warehouse.silver.orders;

SELECT * FROM warehouse.silver.orders LIMIT 10;
```

## 构建 Starrocks 异步物化视图


## Copy Data to Starrocks
```bash