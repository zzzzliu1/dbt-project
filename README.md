<img width="1255" height="551" alt="image" src="https://github.com/user-attachments/assets/ca15c54f-c3fb-451a-892e-1cd714379434" /># dbt-project


## ingest dbt dataset into snowflakes
<img width="1272" height="669" alt="e347d4fa22abd593d4fef29da6f4a851" src="https://github.com/user-attachments/assets/6b753349-386f-40c2-9cc2-b86ba99552a3" />


## ingest dbt dataset into BigQuery
<img width="1253" height="673" alt="image" src="https://github.com/user-attachments/assets/62563a4b-5335-4c9d-be7b-2113e8c90c9d" />



## create a service account and authenticate into dbt and build out dataset into BigQuery
<img width="1253" height="595" alt="image" src="https://github.com/user-attachments/assets/c42787cd-d88a-4c05-b606-23e5851871a8" />


Successfully built two data models in the target dataset in BigQuery
<img width="1260" height="631" alt="image" src="https://github.com/user-attachments/assets/bc18885b-b355-45a0-9f3e-803a23f27121" />



## Build first model
-Explain what models are in a dbt project.
-Build your first dbt model.
-Explain how to apply modularity to analytics with dbt.
-Modularize your project with the ref function.
-Review a brief history of modeling paradigms.
-Identify common naming conventions for tables.
-Reorganize your project with subfolders.



<img width="1216" height="904" alt="image" src="https://github.com/user-attachments/assets/060db045-3a80-4b7b-bf54-e5cca6e29cc5" />
<img width="1648" height="768" alt="image" src="https://github.com/user-attachments/assets/08f09b6c-f7e3-4cb1-97ac-65714722db40" />
<img width="1280" height="591" alt="image" src="https://github.com/user-attachments/assets/55bfee85-92dd-480a-a45c-8fc0bf1b9d6f" />



# dbt Modularity+ ref()宏Macro+ 自动Lineage（依赖关系）
dbt希望做到什么：所有sql 不要直接依赖raw table。而是依赖已经整理好的staging model。于是整个流程变成：
``` text
RAW Table
|
stg_customers
|
customers

```

而不是：
RAW Table
｜
Customers
这就是：Analytics Engineering

第一步：
创建：stg_jaffle_shop_customers.sql
里面其实只有
``` sql
select *
from `dbt-tutorial.jaffle_shop.customers`

运行以后：
dbt会生成：
```
stg_jaffle_shop_customers
```
这个model。

可以理解成：
``` text
Raw Customers

↓

Staging Customers
```

同理：再创建
```
stg_jaffle_shop_orders.sql
```
里面：
```sql
select *
from `dbt-tutorial.jaffle_shop.orders`

于是：
又生成：
```
stg_jaffle_shop_orders
```
于是：
整个project 已经有两个Model。
<img width="603" height="131" alt="image" src="https://github.com/user-attachments/assets/15eeba09-6d9c-4197-9ce5-5e9f0a1be63a" />



第二步：
```
customers.sql
```
不再：直接读取RAW table

而改成：
```sql
select *

from {{ ref('stg_jaffle_shop_customers') }}
```
以及：
```sql
select *

from {{ ref('stg_jaffle_shop_orders') }}
```

## ref 到底干了什么？
第一件事情：自动找到真正的table
例如compiled sql
```sql
select *

from
`jaffle-shop-500907.dbt_cliu.stg_jaffle_shop_orders`
```
所以你写的时候根本不用关系：Database、Schema、Project、Dataset。全部：dbt自动完成。

第二件事情： 建立Dependency（依赖关系）
因为当dbt看到：
```sql
ref('stg_jaffle_shop_orders')
```
它知道customers必须等stg_orders跑完。于是自动建立依赖

第三件事情： 自动生成lineage


```text
BigQuery Raw Tables
│
├── customers
└── orders
        │
        ▼
dbt Staging Layer
│
├── stg_jaffle_shop_customers
└── stg_jaffle_shop_orders
        │
        ▼
Business Layer
│
└── customers.sql
        │
        ▼
Power BI / Looker / Tableau
```

这里面需要注意
``` bash
dbt run --select customers
```
<img width="619" height="393" alt="image" src="https://github.com/user-attachments/assets/88c8c711-a560-479b-be91-2a690cb48245" />

``` bash
dbt run --select +customers
```
<img width="618" height="451" alt="image" src="https://github.com/user-attachments/assets/abf7489b-a0e9-49c1-8da3-e7420eebfde9" />

# Building a Kimball dimensional model with dbt
## Kimball的核心思想是什么： 围绕业务过程（Business Process）建模，而不是围绕数据库建模。
例如：不是cusotmers、orders、payments而是先问：公司真正关心什么？
例如：Sales、Orders、Marketing、Inventory、Shipment
这些加：Business Process

## dbt如何实现Kimball？
dbt不是直接建Fact。而是先Staging
整个流程：
<img width="586" height="329" alt="image" src="https://github.com/user-attachments/assets/1a5cbc88-84d6-4e0f-90ca-6112fba444a6" />
Staging：负责清洗
Intermediate：负责Business Logic


## 文章强调不能直接Join Row？
如果直接Raw -》 Dashboard
如果以后Raw字段改了。所有Dashboard全部坏掉。
所以应该：
<img width="615" height="271" alt="image" src="https://github.com/user-attachments/assets/c91f62bb-f312-4d11-9568-a1f5e80d460a" />

# Data Vault
Data Vault（数据保险库） 是一种功能强大的数据建模方法，非常适用于中大型企业级数据仓库，尤其是具有以下特点的项目：
1. 集成多个不断变化的数据源系统（Integration of multiple dynamic source system）

企业通常有很多不同的数据来源，例如：
- CRM
- ERP
- 电商平台
- 支付系统
- 市场营销平台
这些系统的数据结构（Schema）会不断发生变化。
Data Vault非常擅长把这些不同来源的数据统一整合到一个数据仓库中。

2. 长期项目，并且要求敏捷交付（Long-term project with agile delivery requirements）
很多企业的数据仓库项目并不是几个月就结束，而是持续建设几年甚至十几年。
例如业务部门希望下个月新增一个数据源：
Data Vault 可以支持：
* 敏捷开发（Continuous Development）
* 敏捷迭代（Agile Development）
* 不断增加新的业务，而不需要重构整个数据仓库

3. Data Vault会保留所有历史记录
4. 偏向模块化开发，并希望大量自动化
Data Valut的模型结构非常统一。

例如：
每个业务对象都遵循类似模式：
* Hub
* Link
* Satellite

因此很多SQL可以自动生成：
* dbt Macro
* Automate DV
* VaultSpeed
