# Apache Flink && Apache Iceberg Demo

This demo brings up [Apache Flink](https://flink.apache.org/)
with [Apache Iceberg](https://iceberg.apache.org/), together with
[Nessie](https://projectnessie.org/) as an Iceberg transactional catalog
and [MinIO](https://min.io/) as a storage backend.

## Prerequisites

You need to have [Docker installed](https://docs.docker.com/get-docker/) before running this demo.

## Run the demo

Clone this repository, cd into `flink-iceberg-demo` directory, and start up the demo.

```sh
git clone git@github.com:vontikov/flink-iceberg-demo.git
cd flink-iceberg-demo
docker compose up
```

Open a new terminal and run `Flink SQL client`
```sh
docker exec -it flink-sql-client sql-client
```

Now you can create a new table via set of SQL queries

```sql
CREATE CATALOG nessie
WITH (
    'type'='iceberg',
    'catalog-impl'='org.apache.iceberg.nessie.NessieCatalog',
    'uri'='http://nessie:19120/api/v1',
    'ref'='main',
    'io-impl'='org.apache.iceberg.aws.s3.S3FileIO',
    'warehouse' = 's3://warehouse',
    's3.endpoint'='http://minio:9000',
    's3.path-style-access' = 'true'
);

USE CATALOG nessie;

CREATE DATABASE my_db
WITH (
  'foo'='bar'
);

USE my_db;

CREATE TABLE my_table (
    id   BIGINT,
    name STRING,
    age  INT
) PARTITIONED BY (
    age
) WITH (
     'foo'='bar'
);
```

Insert some data into the table
```sql
INSERT INTO my_table
VALUES
  (1, 'Bob',    42),
  (2, 'Alice',  24),
  (3, 'James',  35),
  (4, 'Carter', 57),
  (5, 'Avery',  30);
```

Once the table is populated you can get the results
```sql
SELECT * FROM my_table;
```

Open MinIO UI and explore objects created by the queries in the bucket
`warehouse`
```sh
open http://localhost:9001
```

You can see Nessie catalog here
```sh
open http://localhost:19120
```

And here you can Flink completed tasks
```sh
open http://localhost:8081
```
