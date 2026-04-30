# 鸡蛋筐回收-用户端与司机端接口清单

本文档按当前已上线代码整理，供 Android / iOS 页面联调使用。

当前实现已经不是“单一鸡蛋筐类型”模式，而是 **按押金类型退还**：

- 一个订单可能同时包含多种押金类型，如 `鸡蛋筐 / 豆腐框 / 水桶`
- 用户提交时可一次选择多种类型和数量
- 后端会按类型拆成 **多张押金单**
- 司机端看到的也是 **一张任务对应一种押金类型的一张押金单**

## 1. 通用返回格式

成功示例：

```json
{
  "code": 1,
  "message": "获取成功",
  "data": {}
}
```

失败示例：

```json
{
  "code": 0,
  "message": "订单不存在",
  "data": []
}
```

说明：

- App 统一以 `code == 1` 作为成功判断
- 失败提示优先展示 `message`
- 旧接口内部有些逻辑仍使用 `msg`，但控制器最终对外统一返回 `message`

## 2. 状态定义

### 2.1 押金单状态 `egg_basket_return.status`

| 值 | 含义 |
| --- | --- |
| `1` | 待分配司机 |
| `2` | 待司机取回 |
| `3` | 已取回待财务退款 |
| `4` | 已退款 |
| `5` | 退款失败 |
| `6` | 已取消 |

### 2.2 财务状态 `egg_basket_return.finance_status`

| 值 | 含义 |
| --- | --- |
| `1` | 未到财务节点 |
| `2` | 待退款 |
| `3` | 已退款 |
| `4` | 退款失败 |

### 2.3 骑手任务状态 `rider_order.status`

| 值 | 含义 |
| --- | --- |
| `1` | 待接单 |
| `2` | 已接单 / 待完成 |
| `3` | 已完成 |

### 2.4 骑手任务类型 `rider_order.type`

| 值 | 含义 |
| --- | --- |
| `1` | 配送单 |
| `2` | 售后退货单 |
| `3` | 押金回收单 |

### 2.5 状态关系说明

用户提交回收申请后，系统会尝试自动匹配“原配送司机”：

- 匹配到司机时：
  - `egg_basket_return.status = 2`，即押金单已进入“待司机取回”
  - 同时生成 `rider_order(type=3)` 任务
  - 新生成的 `rider_order.status = 1`，司机端还需要手动点击“接单”
- 未匹配到司机时：
  - `egg_basket_return.status = 1`
  - 后续由后台人工分配司机

所以要注意：

- **押金单状态** 和 **司机任务状态** 不是同一个状态机
- 用户端列表以 `egg_basket_return.status` 为准
- 司机端任务列表以 `rider_order.status` 为准

## 3. 移动端页面实现建议

### 3.1 用户端页面

推荐至少包含以下页面：

1. 可退押金订单列表
2. 提交回收申请页
3. 我的押金单列表
4. 押金单详情页

关键实现：

- 订单卡片上展示 `deposit_types`
- 每种押金类型一行，支持单独输入数量
- 提交时只上传数量大于 `0` 的类型
- 一次提交多类型时，后端会拆成多张押金单，列表页需按“多条记录”展示

### 3.2 司机端页面

推荐至少包含以下页面：

1. 回收任务列表
2. 任务详情
3. 接单
4. 上传回收照片
5. 提交回收结果

关键实现：

- `orders?order_type=3` 拉的每一条都是当前司机自己的任务
- 一条任务对应一张押金单
- 一张押金单当前只对应一种 `deposit_type`

## 4. 用户端接口

## 4.1 可退押金订单列表

- 路径：`/api/Basket/getEligibleBasketReturnOrders`
- 方法：`GET`
- 鉴权：需要用户 `Token`
- 用途：列出当前用户 **仍有可退数量** 的已完成订单；每条记录中业务字段与 `getOrderBasketReturnInfo` 成功时 **单条 `data` 结构一致**（含 `deposit_types`、`goods`、联系人、地址等），并额外包含 `shop_id`、`shop_name`、`created_time`、`complete_time`（Unix 秒），便于列表页直接展示。
- 排序：按订单 `complete_time` 倒序，其次 `id` 倒序。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `page` | int | 否 | 页码，默认 `1` |
| `page_size` | int | 否 | 每页条数，默认 `10`，最大 `50` |

成功返回 `data` 结构（与 ThinkPHP 分页字段对齐，便于列表组件使用）：

```json
{
  "data": [
    {
      "order_id": 12345,
      "order_no": "S202604140001",
      "shop_id": 1,
      "shop_name": "西湖鲜蛋铺",
      "created_time": 1776000000,
      "complete_time": 1776100000,
      "status": 2,
      "status_name": "待司机取回",
      "deposit_types": [
        {
          "deposit_type": 1,
          "name": "鸡蛋筐",
          "max_basket_count": 2,
          "unit_deposit_price": "15.00",
          "max_refund_amount": "30.00",
          "cover": "https://oss.example.com/img/20260429/egg.jpg",
          "order_goods_id": 1001,
          "goods_id": 501,
          "goods_name": "鲜鸡蛋",
          "sku_title": "30枚/框"
        },
        {
          "deposit_type": 2,
          "name": "豆腐框",
          "max_basket_count": 1,
          "unit_deposit_price": "10.00",
          "max_refund_amount": "10.00",
          "order_goods_id": 1002,
          "goods_id": 502,
          "goods_name": "豆腐",
          "sku_title": "整箱",
          "cover": "https://oss.example.com/img/20260429/tofu.jpg"
        }
      ],
      "eligible_basket_count": 3,
      "unit_deposit_price": "15.00",
      "max_refund_amount": "40.00",
      "contact_name": "张三",
      "contact_tel": "13800000000",
      "pickup_address_id": 88,
      "pickup_address": "杭州市西湖区古墩路xxx"
    }
  ],
  "total": 1,
  "per_page": 10,
  "current_page": 1,
  "last_page": 1,
  "meta": {
    "empty_reason": null,
    "completed_order_count": 5
  }
}
```

字段说明（与 `getOrderBasketReturnInfo` 单条 `data` 一致）：

- **`goods`**：订单维度扁平列表，便于列表页一行展示多商品。
- **`deposit_types[].goods`**：按押金类型分组后的商品明细，与单订单接口相同。
- 每条商品明细含 **`cover`**：主图完整 URL；优先订单行 `order_goods.cover` 快照，否则取 `product.cover` 逗号分隔中的第一张；无图为空字符串 `""`。

`meta.empty_reason`（仅当 **`total === 0`** 时有意义，用于区分空列表文案）：

| 值 | 含义（建议前端文案方向） |
| --- | --- |
| `no_completed_orders` | 没有「已完成且已支付」的订单 |
| `no_deposit_container` | 有已完成订单，但订单内没有标记为押金商品（`product.is_deposit=1`）的行 |
| `nothing_left` | 有押金商品，但当前已无可退数量（均已申请/占用完毕） |

当 `total > 0` 时，`empty_reason` 为 `null`。

说明：

- 列表中的 `deposit_types` / `goods` 与单订单接口一致，申请页可直接复用或再调 `getOrderBasketReturnInfo` 二次确认。
- `completed_order_count` 为「已完成且已支付」订单总数，用于与「可退条数为 0」对照展示。

## 4.2 获取订单可退押金信息

- 路径：`/api/Basket/getOrderBasketReturnInfo`
- 方法：`GET`
- 鉴权：需要用户 `Token`
- 用途：进入“申请回收”页前，先查询订单当前可退的押金类型、每种类型上限、联系人和取货地址。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `order_id` | int | 是 | 订单 ID |

成功返回 `data` 示例：

```json
{
      "order_id": 12345,
      "order_no": "S202604140001",
      "shop_id": 1,
      "shop_name": "西湖鲜蛋铺",
      "created_time": 1776000000,
      "complete_time": 1776100000,
      "deposit_types": [
        {
          "deposit_type": 1,
          "name": "鸡蛋筐",
          "max_basket_count": 2,
          "unit_deposit_price": "15.00",
          "max_refund_amount": "30.00",
          "cover": "https://oss.example.com/img/20260429/egg.jpg",
          "order_goods_id": 1001,
          "goods_id": 501,
          "goods_name": "鲜鸡蛋",
          "sku_title": "30枚/框"
        },
        {
          "deposit_type": 2,
          "name": "豆腐框",
          "max_basket_count": 1,
          "unit_deposit_price": "10.00",
          "max_refund_amount": "10.00",
          "order_goods_id": 1002,
          "goods_id": 502,
          "goods_name": "豆腐",
          "sku_title": "整箱",
          "cover": "https://oss.example.com/img/20260429/tofu.jpg"
        }
      ],
      "eligible_basket_count": 3,
      "unit_deposit_price": "15.00",
      "max_refund_amount": "40.00",
      "contact_name": "张三",
      "contact_tel": "13800000000",
      "pickup_address_id": 88,
      "pickup_address": "杭州市西湖区古墩路xxx"
    }
```

字段说明：

- `deposit_types`：**申请页主渲染字段**
  - 一个元素表示一种押金类型
  - 前端应根据这个数组渲染多行数量输入框
- `deposit_type`
  - 押金类型 ID，提交时原样回传
- `max_basket_count`
  - 当前订单该类型还可申请的最大数量
- `goods`
  - 该类型对应的商品明细
- `goods[].cover`
  - 商品主图完整 URL；优先 `order_goods.cover`，否则 `product.cover` 首张；无图为 `""`
- `eligible_basket_count`
  - 全部类型合计还可退数量，仅适合做摘要，不适合做具体提交校验
- `unit_deposit_price` / `max_refund_amount`
  - 顶层字段是汇总展示用途
  - 多类型页面请优先使用 `deposit_types[*].unit_deposit_price`

常见失败返回：

- `缺少订单ID`
- `订单不存在`
- `订单未支付`
- `订单未完成，暂不能退押金容器`
- `当前订单无可退押金`

## 4.3 提交押金回收申请

- 路径：`/api/Basket/createBasketReturn`
- 方法：`POST`
- 鉴权：需要用户 `Token`
- 用途：按订单提交押金回收申请，支持单类型和多类型一起提交。

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `order_id` | int | 是 | 订单 ID |
| `returns` | string / array | 推荐 | 多类型提交内容。推荐传 JSON 字符串 |
| `deposit_type` | int | 否 | 兼容旧版单类型提交时使用 |
| `basket_count` | int | 否 | 兼容旧版单类型提交时使用 |
| `contact_name` | string | 是 | 联系人，不传则默认取订单地址联系人 |
| `contact_tel` | string | 是 | 联系电话，不传则默认取订单地址电话 |
| `pickup_address_id` | int | 否 | 取货地址 ID |
| `pickup_address` | string | 是 | 取货地址文本，不传则默认取订单地址 |
| `appointment_time` | int | 否 | 预约取货时间，Unix 时间戳 |
| `remark` | string | 否 | 用户备注 |

### `returns` 推荐格式

推荐传 JSON 字符串：

```json
[
  {
    "deposit_type": 1,
    "basket_count": 2
  },
  {
    "deposit_type": 2,
    "basket_count": 1
  }
]
```

前端提交规则：

- 只提交 `basket_count > 0` 的类型
- `deposit_type` 必须来自 `getOrderBasketReturnInfo.deposit_types`
- 不要把所有类型都提交为 `0`

### 成功返回 `data` 示例

```json
{
  "returns": [
    {
      "return_id": 66,
      "return_no": "BK202604141430001234",
      "deposit_type": 1,
      "status": 2,
      "status_name": "待司机取回",
      "rider_id": 9,
      "rider_name": "张师傅"
    },
    {
      "return_id": 67,
      "return_no": "BK202604141430001235",
      "deposit_type": 2,
      "status": 2,
      "status_name": "待司机取回",
      "rider_id": 9,
      "rider_name": "张师傅"
    }
  ],
  "return_id": 66,
  "return_no": "BK202604141430001234",
  "status": 2,
  "status_name": "待司机取回",
  "max_refund_amount": "3",
  "rider_id": 9,
  "rider_name": "张师傅",
  "assign_notice": "已自动分配原配送司机"
}
```

返回说明：

- 一次多类型提交，后端会拆成多张押金单
- `returns` 是本次提交创建的全部押金单
- 顶层 `return_id / return_no / status / rider_id / rider_name` 只是第一张押金单的信息，主要为了兼容旧客户端
- 新客户端请以 `returns` 为准

状态说明：

- 匹配到原配送司机时，返回 `status = 2`
- 未匹配到司机时，返回 `status = 1`
- `assign_notice` 可直接展示给用户

常见失败返回：

- `缺少订单ID`
- `订单不存在`
- `订单未支付`
- `订单未完成，暂不能退押金容器`
- `当前订单无可退押金`
- `请指定押金类型与退框数量`
- `订单不包含该押金类型或已无可退数量`
- `申请数量超过该类型可退上限`
- `押金类型无效或已停用`
- `联系人、电话、取货地址不能为空`

## 4.4 获取押金单列表

- 路径：`/api/Basket/getBasketReturnList`
- 方法：`GET`
- 鉴权：需要用户 `Token`
- 用途：用户查看自己的押金回收记录。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `status` | int | 否 | 按押金单状态筛选 |
| `page_size` | int | 否 | 每页条数，默认 `10` |

返回说明：

- `data` 为 ThinkPHP 分页对象
- 重点字段是 `data.data` 数组
- 一次多类型提交会在列表里看到多条记录

列表单项示例：

```json
{
  "id": 66,
  "return_no": "BK202604141430001234",
  "uid": 10086,
  "source_order_id": 12345,
  "source_order_no": "S202604140001",
  "deposit_type": 1,
  "contact_name": "张三",
  "contact_tel": "13800000000",
  "pickup_address": "杭州市西湖区古墩路xxx",
  "pickup_address_id": 88,
  "appointment_time": 1776178800,
  "apply_basket_count": 2,
  "actual_basket_count": 2,
  "unit_deposit_price": "15.00",
  "apply_refund_amount": "30.00",
  "actual_refund_amount": "30.00",
  "status": 3,
  "pickup_status": 3,
  "finance_status": 2,
  "rider_id": 9,
  "rider_name": "张师傅",
  "pickup_images": "https://xxx/a.jpg,https://xxx/b.jpg",
  "pickup_images_list": [
    "https://xxx/a.jpg",
    "https://xxx/b.jpg"
  ],
  "driver_remark": "【司机：张师傅】\n已回收2框",
  "user_remark": "放门口即可",
  "created_time": 1776175200,
  "created_time_text": "2026-04-14 14:00:00",
  "appointment_time_text": "2026-04-14 18:00:00",
  "finance_time_text": "",
  "status_name": "已取回待财务退款",
  "finance_status_name": "待退款"
}
```

页面展示建议：

- 列表页展示 `return_no`
- 展示 `apply_basket_count`
- 展示 `apply_refund_amount`
- 展示 `status_name`
- 展示 `finance_status_name`
- `deposit_type` 建议前端本地映射名称：
  - `1=鸡蛋筐`
  - `2=豆腐框`
  - `3=水桶`

补充说明：

- 当前用户列表接口不保证返回 `deposit_type_name`
- 如需要类型名，前端请基于 `deposit_type` 本地映射

## 4.5 获取押金单详情

- 路径：`/api/Basket/getBasketReturnDetail`
- 方法：`GET`
- 鉴权：需要用户 `Token`
- 用途：查看单个押金单详情。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `return_id` | int | 是 | 押金单 ID |

成功返回说明：

- 主信息字段基本与列表单项一致
- 额外返回 `items` 明细数组

`items` 单项示例：

```json
{
  "id": 1,
  "return_id": 66,
  "order_id": 12345,
  "order_goods_id": 1001,
  "goods_id": 501,
  "goods_name": "鲜鸡蛋",
  "sku_title": "30枚/框",
  "eligible_basket_count": 2,
  "applied_basket_count": 2,
  "actual_basket_count": 2,
  "unit_deposit_price": "15.00",
  "applied_refund_amount": "30.00",
  "actual_refund_amount": "30.00"
}
```

常见失败返回：

- `缺少回收单ID`
- `回收单不存在`

## 4.6 取消押金单

- 路径：`/api/Basket/cancelBasketReturn`
- 方法：`POST`
- 鉴权：需要用户 `Token`
- 用途：取消尚未进入司机处理的申请。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `return_id` | int | 是 | 押金单 ID |

说明：

- 只有 `status = 1` 的押金单允许取消
- 已自动分配司机、已进入回收流程的单子不能取消

成功返回：

```json
{
  "code": 1,
  "message": "取消成功",
  "data": []
}
```

常见失败返回：

- `缺少回收单ID`
- `回收单不存在`
- `当前状态不可取消`

## 5. 司机端接口

## 5.1 获取任务列表

- 路径：`/api/Rider/orders`
- 方法：`GET`
- 鉴权：需要司机 `Token`
- 用途：获取当前司机自己的任务列表。押金回收任务请传 `order_type=3`。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `order_status` | int | 否 | `1=待接单`、`2=已接单`、`3=已完成` |
| `order_type` | int | 否 | 押金回收传 `3` |
| `warehouses_id` | int/string | 否 | 仓库 ID |
| `page` | int | 否 | 页码；当前仅第一页有数据，`page > 1` 直接返回空数组 |
| `lat` | string | 是 | 当前司机纬度 |
| `lng` | string | 是 | 当前司机经度 |

成功返回 `data` 示例：

```json
{
  "data": [
    {
      "id": 501,
      "type": 3,
      "order_id": 66,
      "status": 1,
      "order_no": "BK202604141430001234",
      "order_note": "放门口即可",
      "user_address": "杭州市西湖区古墩路xxx",
      "user_tel": "13800000000",
      "shop_name": "",
      "full_name": "张三",
      "warehouse_address": "仓库地址",
      "warehouse_name": "一号仓",
      "warehouse_tel": "0571xxxxxxx",
      "superintendent": "仓管A",
      "user_distance": "2.10公里",
      "user_longitude": "120.12",
      "user_latitude": "30.28",
      "warehouse_distance": "5.30公里",
      "warehouse_longitude": "120.10",
      "warehouse_latitude": "30.20"
    }
  ]
}
```

重点字段：

- `id`：骑手任务 ID，后续接单 / 查看详情 / 完成回收都传这个值
- `order_id`：押金单 ID
- `order_no`：押金单号
- `status`：骑手任务状态，不是押金单状态
- `order_note`：用户备注

补充说明：

- 如果用户一次提交多类型，且都自动匹配到同一个原配送司机，司机端会看到多条任务
- 每条任务对应一张押金单

常见失败返回：

- `经纬度不可为空`

## 5.2 获取任务详情

- 路径：`/api/Rider/orderDetail`
- 方法：`GET`
- 鉴权：需要司机 `Token`
- 用途：查看单个任务详情。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | int | 是 | 骑手任务 ID |
| `lat` | string | 是 | 当前司机纬度 |
| `lng` | string | 是 | 当前司机经度 |

返回 `data` 示例：

```json
{
  "id": 501,
  "type": 3,
  "order_id": 66,
  "status": 2,
  "deliver_pic": "https://xxx/a.jpg,https://xxx/b.jpg",
  "arrive_time": "",
  "delivery_time": "2026-04-14 15:00:00",
  "order_no": "BK202604141430001234",
  "user_address": "杭州市西湖区古墩路xxx",
  "user_tel": "13800000000",
  "shop_name": "",
  "full_name": "张三",
  "warehouse_address": "仓库地址",
  "warehouse_name": "一号仓",
  "warehouse_tel": "0571xxxxxxx",
  "superintendent": "仓管A",
  "order_note": "放门口即可",
  "apply_basket_count": 2,
  "actual_basket_count": 0,
  "user_distance": "2.10公里",
  "warehouse_distance": "5.30公里",
  "goods": [
    {
      "goods_name": "鲜鸡蛋",
      "sku_title": "30枚/框",
      "num": 2
    }
  ],
  "keybox_password": "123456"
}
```

说明：

- `apply_basket_count` 是这张押金单申请数量
- 当前一张押金单只对应一种押金类型
- 司机端如果要显示押金类型名称，建议结合业务本地映射或由服务端后续补字段

常见失败返回：

- `ID不可为空`
- `经纬度不可为空`

## 5.3 司机接单

- 路径：`/api/Rider/confirmPickup`
- 方法：`POST`
- 鉴权：需要司机 `Token`
- 用途：司机确认接单。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | int | 是 | 骑手任务 ID |

成功返回：

```json
{
  "code": 1,
  "message": "操作成功",
  "data": []
}
```

执行效果：

- 更新 `egg_basket_return.status = 2`
- 更新 `egg_basket_return.pickup_status = 2`
- 更新 `egg_basket_return.rider_id = 当前司机ID`
- 更新 `egg_basket_return.driver_remark = 【司机：司机姓名】`
- 更新 `rider_order.status = 2`

常见失败返回：

- `ID不可为空`
- `操作失败`

## 5.4 提交回收结果

- 路径：`/api/Rider/confirmArrival`
- 方法：`POST`
- 鉴权：需要司机 `Token`
- 用途：司机完成上门回收后提交结果。

请求参数：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | int | 是 | 骑手任务 ID |
| `pic` | string | 是 | 回收照片 URL，多个用英文逗号拼接 |
| `actual_basket_count` | int | 是 | 实际回收数量，必须大于 0，且不能大于申请数量 |
| `remark` | string | 否 | 司机备注 |

成功返回：

```json
{
  "code": 1,
  "message": "操作成功",
  "data": []
}
```

执行效果：

- 更新 `egg_basket_return.status = 3`
- 更新 `egg_basket_return.pickup_status = 3`
- 更新 `egg_basket_return.finance_status = 2`
- 更新 `actual_basket_count`
- 更新 `actual_refund_amount`
- 保存 `pickup_images`
- 写入 `driver_remark`
- 更新对应明细 `egg_basket_return_item.actual_basket_count / actual_refund_amount`
- 更新 `rider_order.status = 3`

说明：

- 实际退款金额由服务端根据该单单价计算
- 多类型不是在一个任务里提交多行，而是每张任务分别提交一次

常见失败返回：

- `ID不可为空`
- `请上传送达图片`
- `押金单不存在`
- `请填写实际取回框数`
- `实际取回框数不能大于申请框数`

## 5.5 上传图片

- 路径：`/api/Upload/uploadImg`
- 方法：`POST`
- 用途：先上传图片，再把返回 URL 传给 `confirmArrival.pic`

请求方式：

- `multipart/form-data`
- 文件字段名：`file`

成功返回示例：

```json
{
  "code": 1,
  "message": "文件上传成功",
  "data": {
    "preview": "https://oss-domain/img/20260414/xxxxxxxx.jpg"
  }
}
```

说明：

- `data.preview` 就是图片 URL
- 多张图时，前端上传多次后把 URL 用英文逗号拼接传给 `pic`

## 6. 推荐调用顺序

### 6.1 用户端

1. 可退押金订单列表页调 `getEligibleBasketReturnOrders`（或进入申请前再调 `getOrderBasketReturnInfo` 二次拉齐）
2. 进入申请页时调 `getOrderBasketReturnInfo`（可选：列表项已含相同结构时可仅在下拉刷新时调用）
3. 根据 `deposit_types` 渲染每种押金类型的输入行
4. 组装 `returns` 后调 `createBasketReturn`
5. 我的回收单列表调 `getBasketReturnList`
6. 详情页调 `getBasketReturnDetail`
7. `status = 1` 时可调 `cancelBasketReturn`

### 6.2 司机端

1. 调 `orders?order_type=3`
2. 进入详情调 `orderDetail`
3. 点击接单调 `confirmPickup`
4. 上传图片调 `uploadImg`
5. 提交回收结果调 `confirmArrival`

## 7. 前端实现注意事项

1. 新客户端请按 **多押金类型** 实现，不要只保留一个 `basket_count` 输入框。
2. 提交时请以 `returns` 为主，`deposit_type + basket_count` 仅视为兼容旧版。
3. 用户端列表接口当前不稳定返回 `deposit_type_name`，请前端本地维护类型名映射。
4. 一次多类型提交后，后端会拆成多张押金单，列表页要按多条数据展示。
5. 司机端任务也是一条任务对应一张押金单，不需要在司机端做多类型合并提交。
