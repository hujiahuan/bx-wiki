# 客户画像表结构设计

> 来源：笨熊进货App_20260423说明用.xlsx - Sheet 5

---

## 概述

千人千面策略的客户画像相关表结构设计方案，包含四个核心表：

---

## 一、核心基础信息表（客户主表）

存储客户的基础身份与经营属性信息，是用户画像的核心基础。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| merchant_id | INT/AUTO_INCREMENT | 客户唯一主键ID，自增 |
| merchant_name | VARCHAR(100) | 客户店铺名称 |
| category | VARCHAR(50) | 经营品类，如早餐店、生鲜店、火锅店等 |
| scale | VARCHAR(20) | 店铺规模，如小型、中型、大型 |
| region | VARCHAR(50) | 所在区域，精确到区/县 |
| register_time | DATETIME | 客户入驻时间 |
| status | TINYINT(1) | 客户状态：0-未激活，1-正常，2-冻结 |

---

## 二、采购行为数据表

记录客户的采购全链路行为，为画像提供动态行为依据。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| record_id | INT/AUTO_INCREMENT | 采购记录主键ID |
| merchant_id | INT | 关联客户主表ID |
| product_id | INT | 采购商品ID |
| product_category | VARCHAR(50) | 采购商品品类 |
| purchase_quantity | INT | 采购数量 |
| purchase_amount | DECIMAL(10,2) | 采购总金额 |
| purchase_time | DATETIME | 采购时间 |
| is_promotion | TINYINT(1) | 是否为促销商品：0-否，1-是 |
| promotion_type | VARCHAR(30) | 促销类型，如满减、折扣、买赠 |

---

## 三、用户画像标签表

基于基础信息与采购行为，生成的标签化画像数据，直接支撑千人千面策略。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| label_id | INT/AUTO_INCREMENT | 标签主键ID |
| merchant_id | INT | 关联客户主表ID |
| label_type | VARCHAR(20) | 标签类型，如基础属性、采购偏好、消费能力 |
| label_name | VARCHAR(50) | 具体标签名称，如早餐品类偏好、高频刚需采购、中高端需求 |
| label_score | DECIMAL(3,2) | 标签匹配度分数，0-1之间，分数越高匹配度越高 |
| update_time | DATETIME | 标签更新时间 |

---

## 四、画像数据更新日志表

记录画像标签的更新历史，便于追溯与模型迭代优化。

| 字段名 | 数据类型 | 说明 |
|-------|---------|------|
| log_id | INT/AUTO_INCREMENT | 日志主键ID |
| merchant_id | INT | 关联客户主表ID |
| old_label | VARCHAR(100) | 更新前的标签集合 |
| new_label | VARCHAR(100) | 更新后的标签集合 |
| update_reason | VARCHAR(100) | 更新原因，如采购行为更新、模型迭代 |
| update_time | DATETIME | 更新时间 |

---

## 五、表结构关联与设计说明

### 主外键关联

采购行为数据表通过 `merchant_id` 关联客户主表，画像标签表和更新日志表也通过 `merchant_id` 关联客户主表，形成完整的客户数据链路。

### 标签动态更新

当采购行为数据表新增或更新数据时，触发画像标签表的标签计算与更新，同时记录到更新日志表，确保画像的实时性。

### 扩展性设计

所有表的字段预留扩展空间，比如标签表的 `label_type` 和 `label_name` 可灵活新增标签类型，适配后续新的千人千面策略需求。