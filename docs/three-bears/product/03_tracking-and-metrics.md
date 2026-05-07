# 埋点及指标

> 来源：笨熊进货App_20260423说明用.xlsx - Sheet 3

---

## 一、核心埋点动作

| 埋点动作 | 埋点目的 | 收集的数据 |
|---------|---------|-----------|
| 首页浏览 | 统计用户活跃度与首页内容吸引力 | user_id, device_id, page_name="home", action_time, network, os |
| 商品搜索 | 分析用户搜索习惯与热门关键词 | user_id, keyword, search_result_count, action_time, page_source |
| 商品点击 | 评估商品曝光转化率与用户兴趣偏好 | user_id, product_id, sku_id, page_source, action_time, position_in_list |
| 加入购物车 | 监控转化漏斗，识别流失节点 | user_id, product_id, sku_id, quantity, price, total_amount, action_time |
| 购物车浏览 | 了解用户决策时长与比价行为 | user_id, cart_item_count, total_value, action_time, duration |
| 提交订单 | 记录交易发起行为，计算下单转化率 | user_id, order_id, total_amount, item_count, action_time, payment_method |
| 支付成功 | 确认最终成交，统计GMV与客单价 | user_id, order_id, final_amount, coupon_used, action_time, is_first_order |
| 优惠券领取 | 评估营销活动吸引力与使用率 | user_id, coupon_id, discount_value, valid_period, action_time, source_channel |
| 个人中心访问 | 分析用户忠诚度与功能使用频率 | user_id, page_name="profile", action_time, visit_duration |
| 推送消息点击 | 衡量消息触达效果与召回能力 | user_id, notification_id, click_source, target_page, action_time |

---

## 二、数据分析指标体系

### 1. 核心转化漏斗

- **指标构成**：首页访问 → 商品点击 → 加入购物车 → 提交订单 → 支付成功
- **分析价值**：识别用户流失关键节点，优化产品路径与交互设计
- **示例看板**：通过漏斗图展示各环节转化率，支持按渠道、用户分群下钻分析

### 2. GMV与订单趋势

- **指标构成**：日/周/月 GMV、订单量、客单价（GMV / 有效订单数）
- **分析价值**：监控整体销售健康度，识别增长或下滑趋势，支撑经营决策
- **示例看板**：折线图 + 柱状图组合，叠加同比/环比变化率，突出异常波动

### 3. 用户活跃与留存

- **指标构成**：DAU/MAU、次日留存率、7日留存率、30日留存率
- **分析价值**：评估产品粘性与用户忠诚度，衡量拉新与促活策略效果
- **示例看板**：热力图展示留存矩阵，结合新老用户分群对比趋势

### 4. 商品表现 TOP 榜

- **指标构成**：商品曝光量、点击率（CTR）、加购转化率、成交转化率
- **分析价值**：识别爆款与滞销品，优化商品排序、推荐策略与库存分配
- **示例看板**：表格形式展示 TOP 20 商品，支持按类目筛选与多维度排序

### 5. 营销活动 ROI

- **指标构成**：优惠券领取率、使用率、券后 GMV 增量、获客成本（CAC）
- **分析价值**：评估促销活动投入产出比，优化预算分配与玩法设计
- **示例看板**：仪表盘展示关键活动指标，联动渠道来源与用户画像维度