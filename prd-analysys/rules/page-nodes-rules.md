## 页面路径地图规范


### 一、顶层结构

```json
{
  "version": "string",
  "updatedAt": "string",
  "source": {
    "prd": "string",
    "prototype": "string"
  },
  "notes": ["string"],
  "apps": [
    {
      "appId": "string",
      "appType": "string",
      "version": "string",
      "nodes": {}
    }
  ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `version` | string | ✅ | 整份页面路径地图数据版本号 |
| `updatedAt` | string | ✅ | 整份地图更新时间，格式 `YYYY-MM-DDTHH:mm:ssZ`。各 App 不再单独维护 `updatedAt` |
| `source` | object | ❌ | 数据来源说明，仅记录 PRD 与原型 |
| `notes` | array[string] | ❌ | 全局说明、合并口径、拆分口径、待确认项等 |
| `apps` | array[object] | ✅ | 多终端 / 多 App 集合 |


---

### 二、App 结构

```json
{
  "appId": "string",
  "appType": "string",
  "version": "string",
  "nodes": {}
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `appId` | string | ✅ | App 唯一标识，如 `com.youyu.miniprogram` |
| `appType` | string | ✅ | 终端类型枚举，见下表 |
| `version` | string | ✅ | 当前 App 节点集合的版本号 |
| `nodes` | object | ✅ | 页面节点集合，key 为 `pageId` |

> `version` 语义区分：顶层 `version` 为整份地图数据版本；App 内 `version` 为该终端节点集合版本。两者独立演进。

---

### 三、appType 枚举值

| 值 | 说明 | 示例 |
|----|------|------|
| `minapp` | 小程序 | 微信小程序、支付宝小程序 |
| `app` | 客户端 | iOS / Android 原生 App |
| `h5` | 网页端 | 移动端 H5、PC 网页 |
| `admin` | 管理后台 | 运营后台、管理后台 |

---

### 四、节点字段定义

单个 App 内的 `nodes` 保持原规范：

```json
{
  "pageId": {
    "pageId": "string",
    "pageName": "string",
    "pageType": "string",
    "path": "string",
    "to": ["string"],
    "tags": ["string"],
    "description": "string"
  }
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `pageId` | string | ✅ | 页面唯一标识，同一 App 内全局唯一 |
| `pageName` | string | ✅ | 页面显示名称 |
| `pageType` | string | ✅ | 页面类型，见枚举值 |
| `path` | string | ✅ | 路由路径，采用 scheme 方式命名，规则见第六节 |
| `to` | array[string] | ✅ | 可直接跳转的目标页面 ID 列表，空数组 `[]` 表示叶子节点 |
| `tags` | array[string] | ❌ | 业务流标签，用于分组筛选，不承载结构属性 |
| `description` | string | ❌ | 页面功能描述 |


---

### 五、pageType 枚举值

| 值 | 说明 | 示例 |
|----|------|------|
| `tab` | 底部 Tab 页 | 首页、我的、积分中心 |
| `normal` | 普通页面 | 商品详情、订单列表、积分流水 |
| `modal` | 弹窗 / 浮层 / 抽屉 | 登录弹窗、后台订单详情抽屉 |
| `webview` | H5 内嵌页 | 活动页面 |
| `native` | 原生特殊页 | 相机页、扫码页、微信原生支付收银台 |
| `external` | 外部跳转 | 唤起第三方 App、跳转外部 scheme |

---

### 六、path scheme 命名规则

`path` 统一采用 `scheme://...` 形式，scheme 的取值规则如下。

| pageType | path scheme | 示例 |
|----------|-------------|------|
| `tab` | `appType` | `minapp://pages/points/center`、`admin://points/rule` |
| `normal` | `appType` | `minapp://pages/points/ledger`、`admin://points/goods` |
| `modal` | `modal` | `modal://login`、`modal://admin/points/orders/detail` |
| `native` | `native` | `native://wxpay/cashier`、`native://camera` |
| `external` | `external` | `external://wechat/launch`、`external://alipay/scan` |
| `webview` | `appType` | `minapp://webview/activity`、`h5://webview/activity` |

规则说明：

1. **站内普通页面**（`tab` / `normal` / `webview`）使用 `appType` 作为 scheme，即 `minapp://`、`app://`、`h5://`、`admin://`。
2. **特殊类型**分别使用 `modal` / `native` / `external` 作为 scheme。
3. `external` 的跳转目标写在 `path` 中，`to` 必须为空数组。
4. `modal` / `native` / `external` 的 `path` 不可为空，必须写明 scheme 与目标。
5. 同一 App 内 `path` 唯一；如出现重复，打警告，由人工确认是否为同一页面的别名。

---

### 七、字段命名规则

| 字段 | 规则 |
|------|------|
| `appId` | 反向域名，全局唯一，如 `com.youyu.miniprogram`、`com.youyu.admin` |
| `appType` | 必须为枚举值之一：`minapp`、`app`、`h5`、`admin` |
| `pageId` | 小写字母开头，仅含小写字母、数字、下划线，正则 `^[a-z][a-z0-9_]*$` |
| `pageName` | 中文或英文均可，无特殊限制 |
| `pageType` | 必须为枚举值之一 |
| `path` | 必须为 `scheme://...` 形式，scheme 取值遵循第六节 |
| `to` | 数组元素必须为同一 App 内已存在的 `pageId` |
| `tags` | 小写字母 + 下划线，如 `["core_flow", "payment"]`；不要放 `tab`、`leaf` 等结构属性 |

---

### 八、节点约束规则

以下规则对 `apps[]` 中每个 App 的 `nodes` 独立生效。

| 编号 | 规则内容 | 级别 |
|------|----------|------|
| N-01 | 同一 App 内 `pageId` 全局唯一，不可重复 | 阻塞 |
| N-02 | 同一 App 内 `to` 数组中的每个值必须对应本 App 内已存在的 `pageId` | 阻塞 |
| N-03 | `to` 中不能包含自身的 `pageId`（禁止自循环） | 阻塞 |
| N-04 | `pageId` 命名符合 `^[a-z][a-z0-9_]*$` | 阻塞 |
| N-05 | `pageType` 必须为预定义枚举值之一 | 阻塞 |
| N-06 | `to` 数组不能出现循环引用（如 A→B→A） | 建议检查 |
| N-07 | 单个 App 页面总数 ≥ 1 | 阻塞 |
| N-08 | `apps` 数组不能为空，且每个 `appId` 全局唯一 | 阻塞 |
| N-09 | `appType` 必须为枚举值之一：`minapp`、`app`、`h5`、`admin` | 阻塞 |
| N-10 | `pageType: "external"` 的节点，`to` 必须为空数组 | 阻塞 |
| N-11 | `path` 必须为 `scheme://...` 形式 | 阻塞 |
| N-12 | `path` 的 scheme 与 `pageType` 的对应关系符合第六节 | 阻塞 |
| N-13 | `modal` / `native` / `external` 的 `path` 不可为空 | 阻塞 |

---

### 九、完整示例

```json
{
  "version": "1.0.0",
  "updatedAt": "2026-09-09T06:47:50Z",
  "source": {
    "prd": "积分商城需求PRD.md",
    "prototype": "积分商城需求原型稿.html"
  },
  "notes": [
    "本文件支持多终端 / 多 App 结构，apps 数组内每个 App 的 nodes 独立校验。",
    "appType 为终端类型枚举：minapp-小程序、app-客户端、h5-网页端、admin-管理后台。",
    "顶层 updatedAt 为整份页面路径地图的更新时间，各 App 不再单独维护 updatedAt。",
  ],
  "apps": [
    {
      "appId": "com.youyu.miniprogram",
      "appType": "minapp",
      "version": "1.0.0",
      "nodes": {
        "points_center": {
          "pageId": "points_center",
          "pageName": "积分中心",
          "pageType": "tab",
          "path": "minapp://pages/points/center",
          "to": ["points_ledger", "points_rules", "sign_in", "points_product_detail"],
          "tags": ["core_entry", "points_asset"],
          "description": "积分中心，展示可用积分、赚积分入口、兑换商品列表。"
        },
        "points_ledger": {
          "pageId": "points_ledger",
          "pageName": "积分明细",
          "pageType": "normal",
          "path": "minapp://pages/points/ledger",
          "to": [],
          "tags": ["points_asset", "flow"],
          "description": "展示积分获得、扣减、冻结、解冻、退回、过期和人工调整记录。"
        },
        "login_modal": {
          "pageId": "login_modal",
          "pageName": "登录引导弹窗",
          "pageType": "modal",
          "path": "modal://login",
          "to": [],
          "tags": ["auth"],
          "description": "未登录访问积分资产相关页面时引导登录。"
        },
        "wechat_pay_cashier": {
          "pageId": "wechat_pay_cashier",
          "pageName": "现金补差支付（微信原生支付）",
          "pageType": "native",
          "path": "native://wxpay/cashier",
          "to": ["exchange_order_detail"],
          "tags": ["payment"],
          "description": "微信原生支付收银台，支付成功后订单转待履约。"
        },
        "contact_service": {
          "pageId": "contact_service",
          "pageName": "联系客服（唤起微信客服）",
          "pageType": "external",
          "path": "external://wechat/customer-service",
          "to": [],
          "tags": ["service"],
          "description": "唤起微信客服会话，跳转目标由 path 承载。"
        }
      }
    },
    {
      "appId": "com.youyu.admin",
      "appType": "admin",
      "version": "1.0.0",
      "nodes": {
        "admin_point_rule": {
          "pageId": "admin_point_rule",
          "pageName": "积分规则配置",
          "pageType": "normal",
          "path": "admin://points/rule",
          "to": [],
          "tags": ["config", "rule"],
          "description": "配置消费积分、会员倍率、签到奖励、积分有效期等。"
        },
        "admin_goods_list": {
          "pageId": "admin_goods_list",
          "pageName": "积分商品管理",
          "pageType": "normal",
          "path": "admin://points/goods",
          "to": ["admin_goods_edit"],
          "tags": ["goods", "config"],
          "description": "积分商品列表，支持查询、新增、编辑。"
        },
        "admin_order_detail": {
          "pageId": "admin_order_detail",
          "pageName": "兑换订单详情（抽屉）",
          "pageType": "modal",
          "path": "modal://admin/points/orders/detail",
          "to": ["admin_point_ledger"],
          "tags": ["order", "after_sale"],
          "description": "展示用户、商品、收货、积分流水、支付、履约与售后信息。"
        }
      }
    }
  ]
}
```

---

### 十、校验规则汇总

| 编号 | 校验项 | 级别 | 对应节点规则 |
|------|--------|------|--------------|
| V-01 | `apps` 非空，至少包含 1 个 App | 阻塞 | N-08 |
| V-02 | 每个 App 的 `appId` 全局唯一 | 阻塞 | N-08 |
| V-03 | 每个 App 的 `appType` 为枚举值之一：`minapp`、`app`、`h5`、`admin` | 阻塞 | N-09 |
| V-04 | 每个 App 的 `nodes` 非空，至少包含 1 个页面 | 阻塞 | N-07 |
| V-05 | 同一 App 内所有 `pageId` 唯一 | 阻塞 | N-01 |
| V-06 | 同一 App 内所有 `pageId` 命名符合 `^[a-z][a-z0-9_]*$` | 阻塞 | N-04 |
| V-07 | 同一 App 内所有 `pageType` 为枚举值之一 | 阻塞 | N-05 |
| V-08 | 同一 App 内所有 `to` 引用的 `pageId` 在本 App 的 `nodes` 中存在 | 阻塞 | N-02 |
| V-09 | 无自循环（A → A） | 阻塞 | N-03 |
| V-10 | 无循环引用（A → B → A） | 警告 | N-06 |
| V-11 | `pageType` 为 `tab` 或 `normal` 的节点，若 `to` 为空数组，提示确认是否为预期叶子节点 | 警告 | — |
| V-12 | `tags` 数组中元素以小写字母 + 下划线命名，且不含 `tab`、`leaf` 等结构属性 | 建议 | — |
| V-13 | `pageType: "external"` 的节点，`to` 为空数组 | 阻塞 | N-10 |
| V-14 | 所有 `path` 为 `scheme://...` 形式 | 阻塞 | N-11 |
| V-15 | `path` 的 scheme 与 `pageType` 对应关系符合第六节 | 阻塞 | N-12 |
| V-16 | `modal` / `native` / `external` 的 `path` 不为空 | 阻塞 | N-13 |
| V-17 | 同一 App 内 `path` 建议唯一，重复时打警告 | 警告 | — |

---

### 十一、快速开始模板

```json
{
  "version": "1.0.0",
  "updatedAt": "2026-09-09T00:00:00Z",
  "source": {
    "prd": "",
    "prototype": ""
  },
  "notes": [],
  "apps": [
    {
      "appId": "com.your.miniprogram",
      "appType": "minapp",
      "version": "1.0.0",
      "nodes": {
        "page_a": {
          "pageId": "page_a",
          "pageName": "页面A",
          "pageType": "tab",
          "path": "minapp://page/a",
          "to": ["page_b"],
          "tags": ["core_entry"],
          "description": "页面A描述"
        },
        "page_b": {
          "pageId": "page_b",
          "pageName": "页面B",
          "pageType": "normal",
          "path": "minapp://page/b",
          "to": [],
          "tags": ["flow"],
          "description": "页面B描述"
        }
      }
    },
    {
      "appId": "com.your.admin",
      "appType": "admin",
      "version": "1.0.0",
      "nodes": {
        "admin_home": {
          "pageId": "admin_home",
          "pageName": "后台首页",
          "pageType": "normal",
          "path": "admin://home",
          "to": [],
          "tags": ["admin"],
          "description": "后台首页"
        }
      }
    }
  ]
}
```
