# Перенос данных из Excel (1С) в PostgreSQL

## Шаг 1. Подготовить файлы

1.  В каждом Excel-файле сделайте один лист = одна таблица.\
2.  Экспортируйте каждый лист в CSV (UTF-8, разделитель ",", с
    заголовком).
    -   Даты → формат YYYY-MM-DD\
    -   Десятичные числа → точка `.`\
    -   Пустые значения → пустая ячейка

## Шаг 2. Создать базу и пользователя

``` sql
CREATE DATABASE ds_sanitary;
CREATE USER ds_user WITH PASSWORD 'strong_password';
GRANT ALL PRIVILEGES ON DATABASE ds_sanitary TO ds_user;
```

Подключение:

``` bash
psql -h <host> -U ds_user -d ds_sanitary
```

## Шаг 3. Создать схемы и таблицы

``` sql
CREATE SCHEMA staging;
CREATE SCHEMA core;

CREATE TABLE staging.sales (
  order_dt      date,
  partner_code  text,
  sku           text,
  qty           numeric,
  net_price     numeric,
  discount      numeric,
  promo_flag    boolean,
  region        text,
  channel       text,
  returned_qty  numeric
);
```

Пример таблиц в `core`:

``` sql
CREATE TABLE core.dim_partner (...);
CREATE TABLE core.dim_product (...);
CREATE TABLE core.fct_sales (...);
```

## Шаг 4. Загрузка CSV → staging

``` sql
\copy staging.sales FROM '/path/sales.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8');
\copy staging.products FROM '/path/products.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8');
\copy staging.partners FROM '/path/partners.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8');
```

## Шаг 5. Очистка и маппинг (staging → core)

``` sql
INSERT INTO core.dim_partner (partner_code, inn, name, channel, region)
SELECT DISTINCT trim(partner_code), null, null, lower(channel), trim(region)
FROM staging.sales
WHERE partner_code IS NOT NULL
ON CONFLICT (partner_code) DO NOTHING;
```

Пример загрузки фактов:

``` sql
INSERT INTO core.fct_sales (...)
SELECT s.order_dt, dp.partner_id, dpr.product_id, s.qty, s.net_price, s.discount, s.promo_flag, s.region, s.channel, COALESCE(s.returned_qty,0)
FROM staging.sales s
JOIN core.dim_partner dp ON dp.partner_code = trim(s.partner_code)
JOIN core.dim_product dpr ON dpr.sku = trim(s.sku);
```

## Шаг 6. Индексы и права

``` sql
CREATE INDEX ON core.fct_sales(order_dt);
CREATE INDEX ON core.fct_sales(partner_id);
CREATE INDEX ON core.fct_sales(product_id);
```

## Шаг 7. Проверки качества

``` sql
SELECT COUNT(*) FROM core.fct_sales WHERE net_price <= 0;
SELECT sku, COUNT(*) FROM core.dim_product GROUP BY sku HAVING COUNT(*)>1;
```

## Альтернатива: pgAdmin (GUI)

-   Создайте БД и таблицы через Query Tool.\
-   Импортируйте CSV через Import/Export Data.

## Автоматизация через Python

``` python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://ds_user:strong_password@host:5432/ds_sanitary")

df = pd.read_csv("sales.csv", dtype=str)
df["order_dt"] = pd.to_datetime(df["order_dt"]).dt.date
for col in ["qty","net_price","discount","returned_qty"]:
    if col in df.columns:
        df[col] = pd.to_numeric(df[col], errors="coerce")

df.to_sql("sales", engine, schema="staging", if_exists="append", index=False, method="multi", chunksize=5000)
```

## Подводные камни

-   Кодировка: используйте UTF-8.\
-   Десятичные разделители: точка.\
-   Даты: YYYY-MM-DD.\
-   Сначала загружаем измерения, затем факты.\
-   Ограничения (FK/NOT NULL) включаем после проверки staging.
