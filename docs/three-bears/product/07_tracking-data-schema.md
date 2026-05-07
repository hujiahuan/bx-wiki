# 埋点数据存储表结构

> 来源：笨熊进货App_20260423说明用.xlsx - Sheet 7

---

## 一、核心埋点数据主表（ODS层原始数据）

存储未经清洗的原始埋点数据，保留完整行为信息，用于数据追溯与异常排查。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| log_id | INT | 埋点日志主键ID，自增，**主键** |
| merchant_id | VARCHAR(50) | 商家唯一ID，关联商家主表，采用MD5加密存储 |
| event_name | VARCHAR(50) | 事件名称，如 `add_cart`、`submit_order`、`search_keyword`，统一采用小写英文命名 |
| event_time | DATETIME | 事件发生时间，精确到毫秒，采用UTC时间存储 |
| network_type | VARCHAR(20) | 网络类型，如 WiFi、4G、5G，默认值为"unknown" |
| device_info | JSON | 设备信息，包含设备型号、系统版本、设备ID（脱敏后） |
| raw_params | JSON | 原始埋点参数，以JSON格式存储 |
| ip_address | VARCHAR(45) | 商家IP地址，支持IPv4和IPv6 |
| location | VARCHAR(100) | 商家地理位置信息，精确到区县 |
| session_id | VARCHAR(64) | 会话ID，用于识别同一商家的连续操作，采用UUID生成 |

---

## 二、结构化明细数据表（DWD层清洗后数据）

对ODS层数据清洗、标准化后存储，为后续分析提供高质量结构化数据。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| record_id | INT | 明细数据主键ID，自增，**主键** |
| merchant_id | VARCHAR(50) | 关联商家主表ID |
| event_category | VARCHAR(30) | 事件所属类别，如采购、搜索、浏览、售后 |
| event_name | VARCHAR(50) | 标准化后的事件名称 |
| event_time | DATETIME | 事件发生时间 |
| product_id | VARCHAR(50) | 涉及商品ID，非商品相关事件可为空 |
| product_category | VARCHAR(50) | 涉及商品品类 |
| keyword | VARCHAR(100) | 搜索关键词，进行关键词标准化处理 |
| order_amount | DECIMAL(10,2) | 订单金额，仅采购事件填写 |
| payment_type | VARCHAR(20) | 支付方式，如微信、支付宝、银联 |
| region | VARCHAR(50) | 收货区域，仅采购事件填写 |
| recommend_position | INT | 推荐位序号，仅浏览推荐事件填写 |
| stay_duration | DECIMAL(10,3) | 页面停留时长，单位为秒 |
| click_depth | INT | 点击深度 |

---

## 三、商家行为汇总表（DWS层汇总数据）

按商家维度、时间维度汇总核心行为指标，用于快速查询商家长期行为偏好。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| summary_id | INT | 汇总数据主键ID，自增，**主键** |
| merchant_id | VARCHAR(50) | 关联商家主表ID |
| stat_date | DATE | 统计日期，按天汇总，格式为 YYYY-MM-DD |
| stat_month | VARCHAR(7) | 统计月份，格式为 YYYY-MM |
| stat_year | INT | 统计年份 |
| purchase_count | INT | 采购行为次数 |
| purchase_amount | DECIMAL(10,2) | 采购总金额 |
| purchase_frequency | DECIMAL(10,2) | 采购频次（日均采购次数） |
| search_count | INT | 搜索行为次数 |
| search_keyword_count | INT | 不同搜索关键词数量 |
| search_frequency | DECIMAL(10,2) | 搜索频次 |
| browse_count | INT | 浏览行为次数 |
| recommend_click_count | INT | 推荐商品点击次数 |
| recommend_click_rate | DECIMAL(10,2) | 推荐商品点击率 |
| avg_stay_duration | DECIMAL(10,3) | 商品详情页平均停留时长 |
| avg_order_amount | DECIMAL(10,2) | 平均订单金额 |
| max_order_amount | DECIMAL(10,2) | 最高订单金额 |
| min_order_amount | DECIMAL(10,2) | 最低订单金额 |

---

## 四、商家画像标签表（ADS层应用数据）

基于汇总数据生成的标签化画像，直接支撑千人千面策略的个性化推荐。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| label_id | INT | 标签主键ID，自增，**主键** |
| merchant_id | VARCHAR(50) | 关联商家主表ID |
| label_type | VARCHAR(20) | 标签类型，如采购偏好、搜索偏好、消费能力、区域偏好 |
| label_name | VARCHAR(50) | 具体标签名称，如高频采购生鲜、关注火锅食材、中高端消费 |
| label_score | DECIMAL(3,2) | 标签匹配度分数，0-1之间