# Полное руководство по внедрению Data Science & Machine Learning в компании Иберис Групп

> Контекст: "Итальянские" бренды Cezares и BelBagno, производство: Китай/Россия/Европа, продажи — только по РФ через партнёров (розничные магазины, интернет‑магазины, маркетплейсы; ≤5% — комплектовщики, дизайн‑студии, строители). Компания не ведёт прямую розницу. Компания регулирует РРЦ (минимальные/рекомендованные розничные цены).

---

## 1) Бизнес‑цели и ключевые KPI

**Стратегические цели**
- Рост выручки и доли полки у партнёров при соблюдении РРЦ.
- Снижение дефицитов (OOS) и «заморозки» капитала в запасах.
- Повышение маржинальности SKU/каналов.
- Удержание и рост среднего чека партнёров.
- Ускорение оборота запасов и улучшение SLA по отгрузкам.

**Операционные KPI**
- \[Спрос/прогноз\] sMAPE/WAPE по SKU×регион, MAPE на горизонтах 1–12 недель/месяцев.
- \[Запасы\] оборачиваемость, GMROI, сервис‑левел по классам ABC, доля OOS, доля неликвидов.
- \[Продажи\] LTV партнёра, частота закупок, средний чек, отток (% партнёров без закупок N недель/месяцев).
- \[Маркетплейсы\] share of shelf, Buy‑Box share, рейтинг/отзывы, доля карточек с нарушениями РРЦ.
- \[Цены\] эластичность, доля нарушений РРЦ по партнёрам/каналам, скорость выявления/устранения.
- \[Логистика\] точность прогноза времени поставки (lead time), fill‑rate, OTIF.


---

## 2) Карта данных и источники

**Основные источники**
- ERP/1С: заказы, отгрузки, цены, скидки, остатки, закупки, себестоимость, возвраты.
- WMS/TMS: размещение/подбор, отгрузки, маршруты, сроки доставки.
- CRM: карточки партнёров, контакты, коммуникации, задачи.
- PIM/каталог: SKU, атрибуты (бренд, серия, материал, цвет/покрытие, комплектация), упаковка, EAN/GTIN.
- Маркетплейсы/API: цены, наличие, позиции конкурентов, контент карточек, отзывы.
- Финансы: курсы валют (USD/CNY/EUR), пошлины/логистика, доп. расходы (landed cost).
- Внешние календари/сезонность/праздники; события отрасли (по возможности).

**Ключевые справочники и ID**
- **SKU_ID** (стабильный), **GTIN/EAN**, **Brand_ID**, **Series_ID**.
- **Partner_ID** (ретейлер/дистрибьютор/маркетплейс‑вендор), **Channel_ID**, **Region_ID**.
- **Supplier_ID/Factory_ID** (страна производства, MOQ, lead time).
- **PriceList_ID** (РРЦ, оптовые уровни, акции/промо).

**Критично:** единые ключи, версионность цен/атрибутов, иерархии (SKU→модель→серия→категория→бренд).


---

## 3) Модель данных (звёздная схема DWH)

**Измерения (Dimensions)**
- `dim_date` (день/неделя/месяц, праздники).
- `dim_product` (SKU, бренд, серия, категория, атрибуты, упаковка, габариты, country_of_origin).
- `dim_partner` (тип партнёра, канал, регион, ИНН, marketplace_vendor_id).
- `dim_geography` (страна→ФО/регион→город→склад).
- `dim_price` (РРЦ, оптовая, спец.цена, дата вступления, валюта, прайс‑лист).
- `dim_promo` (тип промо, период, механика, бюджет).
- `dim_supplier` (фабрика, MOQ, Incoterms, стандартный/фактический lead time).

**Факты (Facts)**
- `fct_sales` (заказы/отгрузки: qty, net_price, discount, promo_flag, channel).
- `fct_inventory_daily` (остатки, резервы, срок хранения, aging buckets).
- `fct_purchase_orders` (заказы поставщику, ETA/ATA, стоимость, контейнер).
- `fct_returns` (причины, дефекты, гарантия, RMA).
- `fct_prices_observed` (цены по каналам/рынку, выявления нарушений РРЦ).
- `fct_reviews` (тональность, тематики, рейтинг, источник).

**Загрузка**: ELT (raw→staging→dwh→marts). Слои: bronze/silver/gold. Версионность SCD2.


---

## 4) Качество и управление данными (Data Governance)
- **MDM:** уникальные ID, дедупликация партнёров (ИНН/КПП/телефон/домен), SKU‑матчинг.
- **Качество:** правила валидации (обязательные атрибуты, единицы измерения, упаковка), SLA обновлений.
- **Семантика метрик:** единые определения (выручка, маржа, отток, сервис‑левел, OOS).
- **Доступ:** роли (продажи/закупки/маркетинг/финансы), маскирование PII.
- **Data Contracts:** схемы и контракты между источниками и DWH, тесты в dbt.


---

## 5) Прикладные решения Data Science & ML

### 5.1 Прогнозирование спроса (SKU×регион×канал)
**Задача:** план закупок и запасов при длинных поставках из Китая/Европы.
- **Модели:** классика (Prophet, ETS/ARIMA), GBM (LightGBM/XGBoost), нейронные (LSTM/Temporal‑Fusion‑Transformer) для крупных серий; для иерархий — bottom‑up/top‑down/middle‑out, reconciliation (MinT).
- **Интермиттирующий спрос:** Croston/SBA/TSB, ADIDA; бэкап‑модель для «тонких» рядов.
- **Экзогенные признаки:** цена, промо, канал, конкуренты/карточка MP, праздники, погода/сезонность (если релевантно), логистические лаги.
- **Оценка:** WAPE/sMAPE/MAPE, pinball loss для P50/P90. Бэктест с скользящим окном.
- **Выход:** недельные/месячные прогнозы + доверительные интервалы.

### 5.2 Прогнозирование lead time поставщиков
- **Цель:** точность ETA→планирование закупок/контейнеров.
- **Модели:** регрессии/GBM по маршруту, инкотермс, сезонности (фабрика, порт, период), загрузка.
- **Выход:** распределение LT (P50/P90) для расчёта safety stock.

### 5.3 Оптимизация запасов (single/multi‑echelon)
- **Политики пополнения:** (s,S), ROP, периодический обзор. Класс обслуживания на базе ABC/XYZ.
- **Safety Stock:** по вариативности спроса и lead time; целевой service level по группе (A: 95–98%, B: 90–95%, C: 85–90%).
- **Ограничения:** MOQ, кратность коробки, контейнер, бюджет.
- **Оптимизатор:** стохастическое моделирование + целочисленное программирование (PuLP/OR‑Tools).
- **Метрики:** fill‑rate, излишки/неликвиды, оборот, % backorders.

### 5.4 Предиктивный отток партнёров (Churn)
- **Определение:** нет закупок > *X* недель относительно привычного цикла партнёра.
- **Признаки:** RFM, тренды категорий, share‑of‑wallet, изменения ассортимента, SLA отгрузок, нарушения РРЦ, претензии, контакт‑история из CRM.
- **Модели:** логистическая регрессия/GBM/CatBoost; калибровка вероятностей.
- **Выход:** риск‑скор + список «спасательных» действий (персональные офферы в рамках РРЦ, расширение ассортимента, ускоренная отгрузка).

### 5.5 Рекомендательная система B2B
- **Алгоритмы:** item‑item/ALS, association rules (Apriori/FP‑Growth), sequence mining (next‑best‑SKU).
- **Контекст:** тип партнёра/регион/сезон/ценовой уровень/наличие на складе.
- **Витрина:** в CRM/кабинете партнёра/скриптах менеджеров («с этим берут…», бандлы).

### 5.6 Аналитика отзывов и карточек (NLP)
- **Токены:** тональность (sentiment), темы (LDA/BERTopic), извлечение дефектов.
- **Применение:** ТЗ для R&D/QA, доработка упаковки/инструкций, приоритизация карточек.
- **Метрика:** доля негативных тем, скорость устранения.

### 5.7 Мониторинг соблюдения РРЦ (MAP‑подобная задача)
- **Сбор:** API/скрейпинг маркетплейсов и сайтов партнёров.
- **Логика:** матчинг SKU, нормализация цен, учёт купонов/промо.
- **ML:** классификация истинных нарушений vs. шум (ошибки матчинга/временные акции), приоритезация по влиянию на бренд/оборот.
- **Алёрты:** в CRM/телеграм‑бот менеджерам, SLA реакции.

### 5.8 Ценообразование в условиях РРЦ
- **Задача 1:** оптимальная **РРЦ‑лесенка** по сериям/брендам (price architecture).
- **Задача 2:** оптимизация **оптовых скидок/ребейтов** (влияние на закупки при сохранении РРЦ на витрине).
- **ML/ЭКОНОМЕТРИКА:** оценка эластичности по каналам, difference‑in‑differences на промо, байесовские модели спроса; симулятор маржи/доли полки.

### 5.9 Эффективность промо и маркетинга (Uplift)
- **Подход:** uplift‑моделирование/квази‑эксперименты; таргетирование «убеждаемых» партнёров.
- **Метрики:** ITE/Qini/Gini uplift, ROI промо.

### 5.10 Операции склада и логистика
- **Слоттинг:** размещение SKU по ABC/габаритам/частоте заказов (оптимизация путей отбора).
- **Маршрутизация:** OR‑Tools VRP, учёт окон доставки/приоритета клиентов.
- **Аномалии:** детект ошибок отгрузок/возвратов (Isolation Forest/Prophet).

### 5.11 Качество и гарантия
- **Классификация причин брака/возвратов**, кластеризация по фабрикам/сериям.
- **Vendor scorecards:** рейтинг поставщиков по дефектам, задержкам, отклонению спецификаций.

### 5.12 Риски и финансы
- **FX‑риск:** сценарии курсов CNY/USD/EUR → влияние на себестоимость и РРЦ.
- **Полнозатратная себестоимость:** landed cost, разнесение логистики/пошлин.
- **Кредитный риск партнёров:** PD/поведенческие признаки (сроки оплат, споры, тренды заказов).


---

## 6) Витрины данных и дашборды (для принятия решений)
- **Продажи/Клиенты:** RFM, отток, план‑факт, воронка заказов, share‑of‑wallet.
- **Ассортимент:** ABC/XYZ, каннибализация брендов, KVI‑SKU vs long tail, оборачиваемость.
- **Запасы/Закупки:** прогнозы, ROP/SS, неликвиды, aging, сервис‑левел по ABC.
- **Цены/РРЦ:** карта нарушений, реакции партнёров, эластичность, ROI промо.
- **Маркетплейсы:** доля полки, цены/наличие конкурентов, рейтинг/тональность.
- **Операции:** OTIF, fill‑rate, SLA склада/доставки.


---

## 7) Технический стек (пример без привязки к вендору)
- **Хранилище/DWH:** PostgreSQL/ClickHouse/BigQuery/Redshift/Snowflake.
- **Оркестрация:** Airflow, Prefect.
- **Трансформации:** dbt (тесты/документация/контракты).
- **Потоки/интеграции:** Kafka/Kinesis, Fivetran/Airbyte, API коннекторы к MP.
- **ML:** Python, pandas, scikit‑learn, LightGBM/CatBoost/XGBoost, Prophet/Statsmodels, PyTorch (по мере роста).
- **MLOps:** MLflow (эксперименты/registry), DVC, Feast (feature store), Evidently (drift/monitoring).
- **BI:** Power BI/Tableau/Looker Studio; метрики в Metrics Layer (dbt‑metrics/Semantic Layer).
- **Контейнеры/CI/CD:** Docker, GitHub/GitLab CI, ArgoCD.


---

## 8) MLOps и жизненный цикл моделей
1. **Формализация задачи →** дизайн метрик и лейблов (время‑серии — аккуратная CV).
2. **Подготовка признаков →** vitrine/features c версионностью и тестами.
3. **Обучение →** MLflow эксперименты, автоматический подбор гиперпараметров.
4. **Валидация →** бэктесты/TimeSeriesCV, stress‑тесты (OOS всплески, дырки в данных).
5. **Деплой →** batch (ежедневные рекомендации/прогнозы) + API для онлайна (CRM/портал).
6. **Мониторинг →** качество/дрейф/стабильность; алерты; роллбэк.
7. **Ретрейн →** расписание (еженедельно/ежемесячно) + событие‑триггеры (новый сезон, смена РРЦ).


---

## 9) Пошаговый план внедрения (36 месяцев)

**0–6 мес. Подготовка и быстрые победы**
- Аудит источников, MDM, единые ID; слой staging.
- BI‑дашборды по продажам/остаткам/ABC‑XYZ.
- RFM‑сегментация; первые алерты по оттоку (правила/эвристики).

**7–12 мес. Базовые ML‑модули**
- DWH (bronze/silver/gold), автоматизация загрузок.
- Прогноз спроса (MVP) + прогноз lead time (MVP).
- Churn (MVP) с приоритезацией менеджерских задач в CRM.

**13–24 мес. Расширение ML и интеграции**
- Рекомендательная система B2B (batch + онлайн в CRM/портале).
- NLP по отзывам, мониторинг карточек MP.
- Улучшение прогнозов (иерархии, P50/P90), оптимизация запасов и закупок.
- Мониторинг РРЦ и алерты; оценка промо (uplift).

**25–36 мес. Оптимизация и экономика**
- Многоэшелонные запасы, контейнер/MOQ‑оптимизация.
- Динамика оптовых скидок/ребейтов при РРЦ, архитектура цен.
- What‑if симуляторы (FX, логистика, промо); интеграция в S&OP цикл.

**Вехи (go/no‑go):** точность прогнозов ≥ целевых, снижение OOS и неликвидов, рост LTV/ARPU партнёров, сокращение нарушений РРЦ.


---

## 10) Команда и роли
- **Head of Data / Data Product Manager** — приоритизация бэклога, связь с бизнесом.
- **Analytics Engineer** — моделирование витрин/метрик, dbt, тесты.
- **Data Engineer** — пайплайны/интеграции, производительность, SLA.
- **Data Analyst (Sales/SCM/MP)** — дашборды, продуктовая аналитика.
- **ML Engineer** — разработка/деплой моделей, фичестор, мониторинг.
- **BI Developer** — визуализации и метрики в BI.
- **Domain Experts** (закупки, продажи, маркетплейсы, логистика) — владельцы процессов.

**Найм по фазам:** 3–5 человек в год 1; 5–8 к концу года 2; 8–12 в год 3 (по масштабу бизнеса).


---

## 11) Эксперименты и метрики успеха
- **Forecasting:** WAPE/sMAPE по уровням иерархии, service‑level, доля OOS.
- **Churn:** ROC‑AUC/PR‑AUC, top‑N precision, uplift удержания vs. контроль.
- **Recommender:** CTR/Conversion to order, прирост среднего чека/количества SKU в заказе.
- **RRC мониторинг:** время обнаружения→время устранения, влияние на продажи бренда.
- **Inventory:** GMROI, оборот, неликвиды (₽ и %), списания.
- **Маркетплейсы:** Buy‑Box share, рейтинг/тональность, доля карточек с полным контентом.


---

## 12) Риски и меры
- **Данные:** неполнота/дубликаты → MDM/контракты/тесты.
- **Сезонность/шоки:** бэкап‑модели, прогноз с интервалами (P50/P90), ручная корректировка в S&OP.
- **Принятие пользователями:** обучение менеджеров, простые интерфейсы, «советы → действие».
- **Юридика/РРЦ:** корректная трактовка нарушений (купоны/акции), журнал доказательств, справедливая эскалация.


---

## 13) Экономика и ROI (рамочно)
- **Зоны выгоды:** ↓OOS (прирост продаж), ↓неликвидов и списаний, ↓экспедитов, ↑средний чек/частота, ↓утечки РРЦ.
- **Бенчмарки:**
  - −10–30% неликвидов через 12–18 мес,
  - −15–40% OOS на A‑SKU,
  - +3–8% выручки за счёт рекомендаций/удержания,
  - −10–20% ошибок планирования LT.
- **Косты:** команда, DWH/BI/ML‑инфраструктура, данные маркетплейсов, поддержка.


---

## 14) Мини‑рецепты/формулы (практика)
- **Safety Stock (недели):** `z * sqrt(σ_d^2 * L + σ_L^2 * d^2)`
- **Reorder Point:** `d * L + SS`
- **RFM‑скор:** квантильные бины Recency/Frequency/Monetary → 3‑значный код.
- **Churn‑лейбл:** `no_orders_for > k * median_interpurchase_time(partner)`
- **Uplift‑таргетинг:** два дерева (T‑learner) или uplift‑деревья; метрика Qini.


---

## 15) Бэклог ML‑фич (пример)
- Продажи по лагам/скользящим окнам (1–12), тренды, сезонные срезы.
- Цены конкурентов и разница к РРЦ/оптовой.
- Отзывы: топ‑темы/тональность по SKU/серии/бренду.
- Логистика: фактический LT, отклонения vs. план, порт/маршрут.
- Профиль партнёра: тип/размер/регион, доля бренда в категории, соблюдение РРЦ.


---

## 16) Как встроить в процессы S&OP
1. **Demand Review:** прогнозы P50/P90 по SKU×регион + комментарии сейлзов.
2. **Supply Review:** ограничения LT/MOQ/бюджет, оптимизация заказов.
3. **Pre‑S&OP:** сценарии (FX, промо, capacity), компромиссы.
4. **Executive S&OP:** утверждение плана, целевых KPI, рисков.


---


## Приложение A. Примеры схем таблиц (DDL) и SQL

### A1. Справочники (Dimensions)
```sql
-- Товары
CREATE TABLE dim_product (
  product_id        BIGINT PRIMARY KEY,
  sku               TEXT NOT NULL UNIQUE,
  brand_id          BIGINT NOT NULL,
  series_id         BIGINT,
  category          TEXT,
  name              TEXT,
  material          TEXT,
  color             TEXT,
  country_of_origin TEXT,
  gtin              TEXT,
  package_dims_mm   TEXT,
  created_at        TIMESTAMP,
  valid_from        DATE NOT NULL,
  valid_to          DATE,
  is_current        BOOLEAN DEFAULT TRUE
);

-- Партнёры
CREATE TABLE dim_partner (
  partner_id   BIGINT PRIMARY KEY,
  inn          TEXT,
  kpp          TEXT,
  name         TEXT,
  channel      TEXT CHECK (channel IN ('retail','ecom','marketplace','assembler','design','construction')),
  region       TEXT,
  type         TEXT,
  created_at   TIMESTAMP
);

-- Прайс-лист/РРЦ (SCD2)
CREATE TABLE dim_price (
  price_id     BIGINT PRIMARY KEY,
  product_id   BIGINT REFERENCES dim_product(product_id),
  rrc          NUMERIC(12,2), -- РРЦ
  wholesale    NUMERIC(12,2),
  currency     TEXT,
  valid_from   DATE NOT NULL,
  valid_to     DATE,
  is_current   BOOLEAN DEFAULT TRUE
);
```

### A2. Факты (Facts)
```sql
-- Продажи/отгрузки (гранулярность: позиция заказа)
CREATE TABLE fct_sales (
  sales_id     BIGSERIAL PRIMARY KEY,
  order_dt     DATE NOT NULL,
  partner_id   BIGINT REFERENCES dim_partner(partner_id),
  product_id   BIGINT REFERENCES dim_product(product_id),
  qty          NUMERIC(12,3) NOT NULL,
  net_price    NUMERIC(12,2) NOT NULL,
  discount     NUMERIC(12,2) DEFAULT 0,
  promo_flag   BOOLEAN DEFAULT FALSE,
  region       TEXT,
  channel      TEXT,
  returned_qty NUMERIC(12,3) DEFAULT 0
);

-- Ежедневные остатки
CREATE TABLE fct_inventory_daily (
  snapshot_dt DATE NOT NULL,
  product_id  BIGINT REFERENCES dim_product(product_id),
  warehouse   TEXT,
  on_hand_qty NUMERIC(12,3),
  reserved_qty NUMERIC(12,3) DEFAULT 0,
  aging_bucket TEXT,
  PRIMARY KEY (snapshot_dt, product_id, warehouse)
);

-- Наблюдаемые цены с рынка/MP
CREATE TABLE fct_prices_observed (
  observed_dt DATE NOT NULL,
  product_id  BIGINT REFERENCES dim_product(product_id),
  partner_id  BIGINT REFERENCES dim_partner(partner_id),
  listed_price NUMERIC(12,2) NOT NULL,
  currency    TEXT,
  promo_notes TEXT,
  source      TEXT,
  PRIMARY KEY (observed_dt, product_id, partner_id)
);

-- Отзывы
CREATE TABLE fct_reviews (
  review_id   BIGSERIAL PRIMARY KEY,
  review_dt   TIMESTAMP,
  product_id  BIGINT REFERENCES dim_product(product_id),
  source      TEXT,
  rating      INT CHECK (rating BETWEEN 1 AND 5),
  text        TEXT,
  sentiment   NUMERIC(5,2), -- после NLP
  topics      TEXT[]
);
```

### A3. Примеры аналитических запросов (SQL)
```sql
-- 1) Месячные продажи по SKU×регион с маржой
SELECT date_trunc('month', order_dt) AS month,
       p.sku,
       s.region,
       SUM(s.qty) AS units,
       SUM(s.qty * s.net_price) AS revenue
FROM fct_sales s
JOIN dim_product p ON p.product_id = s.product_id
GROUP BY 1,2,3
ORDER BY 1,2,3;

-- 2) Признак оттока партнёра: нет закупок > X дней
WITH last_buy AS (
  SELECT partner_id, MAX(order_dt) AS last_dt
  FROM fct_sales
  GROUP BY partner_id
)
SELECT d.partner_id,
       (CURRENT_DATE - l.last_dt) > INTERVAL '60 days' AS is_churn
FROM dim_partner d
LEFT JOIN last_buy l USING(partner_id);

-- 3) Выявление подозрения на нарушение РРЦ
SELECT o.observed_dt, p.sku, pr.rrc, o.listed_price,
       (o.listed_price < pr.rrc * 0.98) AS rrc_violation
FROM fct_prices_observed o
JOIN dim_product p USING(product_id)
JOIN dim_price pr ON pr.product_id = o.product_id AND pr.is_current
WHERE o.observed_dt = CURRENT_DATE;
```

---

## Приложение B. Псевдокод ключевых ML‑моделей

### B1. Прогнозирование спроса (SKU×регион)
```python
# Input: fct_sales, экзогенные признаки (цены, промо, праздники), частота = неделя
for series in all_series(SKU, region):
    y = build_time_series(series)
    X = build_exogenous(series, price, promo, holidays, channel)
    # baseline
    model1 = Prophet().fit(y, X)
    # gradient boosting
    model2 = LightGBM().fit(lagged_features(y), X)
    # ансамбль
    yhat1 = model1.predict(h=12, X_future)
    yhat2 = model2.predict(h=12, X_future)
    yhat = 0.5*yhat1 + 0.5*yhat2
    save_forecast(series, quantiles(yhat, p=[0.5, 0.9]))
```

### B2. Прогноз lead time поставки
```python
# Обучаем регрессию по маршруту, сезону, фабрике, порту, инкотермс
features = [factory, port_origin, port_dest, month, incoterms, container_load]
model = CatBoostRegressor().fit(features, actual_lead_time)
lt_p50, lt_p90 = predict_quantiles(model, features_new)
```

### B3. Churn prediction партнёров
```python
X = features(RFM, trend_by_category, share_of_wallet, SLA_shipments, RRC_violations,
             support_tickets, price_changes)
y = label_churn(no_orders_for > k * median_cycle(partner))
model = CatBoostClassifier().fit(X_train, y_train)
proba = model.predict_proba(X_score)
alerts = select_top_k_by_gain(proba, k=500)
```

### B4. Рекомендательная система B2B (ALS + правила)
```python
R = build_partner_sku_matrix(qty or revenue)
als = ImplicitALS().fit(R)
rec_als = als.recommend(partner_id, N=20, filters=in_stock & price_tier)
assoc = association_rules(min_support=0.01, min_conf=0.2)
rec_assoc = next_best_items(cart)
recommendations = blend(rec_als, rec_assoc, weights=[0.7, 0.3])
```

### B5. Мониторинг нарушений РРЦ (классификатор)
```python
X = features(price_gap_to_rrc, promo_flag, coupon_detected, seller_rating,
             history_of_violations, competitor_pressure)
y = is_true_violation(labelled)
model = XGBoostClassifier().fit(X, y)
score = model.predict_proba(X_today)
create_alerts(score > threshold_by_impact)
```

---

## Приложение C. Чек‑лист внедрения на 90 дней

### Недели 1–2
- Утвердить бизнес‑KPI и владельцев метрик.
- Провести аудит источников (ERP/CRM/WMS/маркетплейсы), зафиксировать схемы и доступы.
- Определить уникальные ID (SKU, партнёр) и MDM‑правила.

### Недели 3–4
- Поднять DWH (staging) и настроить ежедневные загрузки продаж/остатков/прайсов.
- Сформировать первые BI‑дашборды: продажи, остатки, ABC/XYZ.
- Начать RFM‑сегментацию партнёров (эвристики).

### Недели 5–6
- Завести витрины для прогнозов (sales_weekly, prices, holidays).
- MVP прогноза спроса по A‑категориям (baseline + backtest WAPE).
- Подготовить фичи churn v0 и метрику оттока.

### Недели 7–8
- Развернуть MLflow, настроить трекинг экспериментов.
- MVP churn‑модели (логистическая/CatBoost), выгрузка топ‑рисков в CRM.
- Начать сбор данных о ценах с MP, первичная валидация РРЦ‑кейсов.

### Недели 9–10
- Витрина рекомендаций (partner×sku матрица). Пилот ALS для 10–20 ключевых партнёров.
- Настроить алерты: падение продаж по A‑SKU, OOS‑риск, нарушения РРЦ.

### Недели 11–12
- Ретроспективный S&OP: сравнение прогнозов P50/P90 vs факт, корректировки.
- Подготовить план масштабирования (склады, фабрики, MP) и Roadmap на 6–12 мес.
- Презентация результатов и следующих шагов.

**Критерии успеха 90 дней**
- BI в проде, SLA загрузок ≥ 99%.
- Прогноз A‑SKU WAPE ≤ 25–30% (MVP).
- Топ‑лист партнёров риска оттока и первые win‑back кейсы.
- Запущены алерты OOS/РРЦ, устранены первые нарушения.
```

### Итог
Гайд выше — это полный каркас внедрения DS/ML под специфику оптовой сантехники с брендами и РРЦ: от данных и моделей до процессов, команды и экономического эффекта. Его можно разворачивать по этапам 36‑месячной дорожной карты, начиная с быстрых побед (BI, RFM, ABC/XYZ) и двигаясь к прогнозам, рекомендациям, оптимизации запасов и ценообразованию в рамках РРЦ.

