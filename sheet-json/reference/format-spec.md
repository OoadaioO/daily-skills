# sheet-json v1 格式规范

## 1. 顶层结构

```json
{
  "$schema": "sheet-json/v1",
  "conventions": "financial-model",
  "sheets": [ /* Sheet 对象，至少 1 个 */ ]
}
```

| 字段 | 类型 | 必需 | 说明 |
|---|---|---|---|
| `$schema` | string | ✅ | 固定 `"sheet-json/v1"`，用于格式识别与版本校验 |
| `conventions` | `"financial-model"` | ❌ | 启用财务模型约定：按 §6 规则**自动推断 `role`** 并上色。省略则无颜色 |
| `sheets` | array\<Sheet\> | ✅ | 顺序即工作表顺序 |

**为什么用 envelope 而不是顶层数组**：可版本演进；输入不是本格式时脚本能明确报
「不是 sheet-json v1」，而不是抛出一堆无关解析错误。

---

## 2. Sheet 对象

```json
{
  "name": "需求拆解总表",
  "header": true,
  "columns": ["端", "业务流", "页面", "前端估时"],
  "rows": [
    ["C端", "会员等级与成长值", "会员等级页", "1 天"],
    ["C端", "会员等级与成长值", "成长值明细", "0.5 天"]
  ],
  "merges": [],
  "columnWidths": { "A": 8, "C": 24 },
  "freeze": "A2"
}
```

| 字段 | 类型 | 必需 | 默认 | 说明 |
|---|---|---|---|---|
| `name` | string | ✅ | — | 工作表名，唯一，≤31 字符，不含 `[ ] : * ? / \` |
| `rows` | array\<array\<Cell\>\> | ✅ | — | **数据行，不含表头行** |
| `header` | boolean | ❌ | `true` | 是否把 `columns` 写为第 1 行 |
| `columns` | array\<string\> | `header=true` 时 ✅ | — | 列名，**同时定义列数与列序** |
| `merges` | array\<string\> | ❌ | — | 合并区域，A1 记法，如 `"A1:B1"` |
| `columnWidths` | object\<string, number\> | ❌ | — | 列宽，键为列字母，如 `{"A": 8}` |
| `freeze` | string | ❌ | — | 冻结窗格位置，如 `"A2"`（冻结首行） |

### 关键决策：`columns` 是「列定义」，不是数据

`columns` 承担三重职责：列名文本、列顺序、列数基准。数据行里**不再重复表头**。
这样消除两个歧义：表头算不算数据行、`rows` 某行少一列时该对齐到哪。

---

## 3. 单元格（Cell）

### 3.1 标量形态（覆盖约 95% 场景）

| JSON 值 | 写入 Excel 结果 |
|---|---|
| `"文本"` | 文本单元格 |
| `123`、`1.5` | 数字单元格 |
| `true` / `false` | 布尔单元格 |
| `null` | 空单元格 |

### 3.2 对象形态（特殊单元格）

统一为**一个对象**，按出现的字段生效：

```json
{ "v": 0.15, "preset": "percent" }
{ "f": "SUM(D2:D17)" }
{ "f": "B2/C2", "v": 0.35 }
{ "v": "2026-09-09", "t": "date" }
{ "v": 5000, "role": ["input", "assumption"], "note": "Source: 10-K FY2024 P45" }
{ "v": "点我", "link": "https://example.com" }
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `v` | 标量 | 值。与 `f` 同时出现时表示**读取到的缓存值**（写路径行为见 §4.3） |
| `f` | string | 公式，**不带 `=`**（带了脚本会剥除） |
| `t` | `"date"` \| `"datetime"` \| `"time"` | 将 `v` 按此类型解析为 Excel 日期/时间 |
| `preset` | 见 §5 枚举 | 命名数字格式。与 `fmt` **互斥** |
| `fmt` | string | 字面 Excel 数字格式串，如 `"0.0%"`。与 `preset` 互斥 |
| `role` | string \| array\<string\> | 语义角色，见 §6，决定字体色/底色 |
| `note` | string | 单元格批注，用于标注硬编码来源 |
| `link` | string | 超链接目标；若同时有 `v`，`v` 为显示文本 |
| `array` | boolean | `f` 为数组公式（openpyxl `ArrayFormula`） |

组合规则：

- 有 `f` → 公式单元格（`v` 可选）
- 无 `f`、有 `t` → 日期/时间单元格
- 无 `f`、无 `t` → 普通值单元格，取 `v`
- `preset` / `fmt` / `role` / `note` 可与以上任意组合
- `v` / `f` / `link` 至少要有一个，不允许 `{"preset":"percent"}` 这种空壳



## 4 公式书写约定

- **用公式，不要硬编码计算结果**。合计写 `{"f":"SUM(B2:B10)"}` 而不是把 Python 算出的
  数字塞进去，这样表才是活的。
- **假设放独立单元格再引用**：`=B5*(1+$B$6)` 而非 `=B5*1.05`。
  公式里的魔数会触发 `MAGIC_NUMBER` warning（绝对值 ≤ 12 的整数视为结构性常量，豁免）。
- 支持绝对引用 `$B$6`、跨表 `'明细'!B2`、外部 `[1]Sheet1!A1`。
- 不带前导 `=`（带了会被剥除，容错但非推荐）
- 支持绝对引用 `$B$6`、跨表 `'明细'!B2`、外部 `[1]Sheet1!A1`
- 参考 xlsx 技能「Assumptions Placement」：假设放进独立单元格再引用
- 除法与查找**不强制**包裹 `IFERROR`；是否需要由建模者自行判断，脚本只报告未验证数量，不施加写法约束


---

## 5. 数字格式预设 `preset`

把 xlsx SKILL.md 的 Number Formatting Standards 固化为枚举，AI 无需背格式码。

| `preset` | 实际 Excel 格式串 | 对应 xlsx 技能条款 |
|---|---|---|
| `integer` | `#,##0;(#,##0);-` | 负数括号；零显示 `-` |
| `decimal` | `#,##0.00;(#,##0.00);-` | 同上，两位小数 |
| `currency` | `$#,##0;($#,##0);-` | 「Currency: Use $#,##0」+ 零为 `-` |
| `currency2` | `$#,##0.00;($#,##0.00);-` | 同上，精确到分 |
| `percent` | `0.0%;(0.0%);-` | 「Percentages: Default to 0.0%」+ 零为 `-` |
| `multiple` | `0.0x;(0.0x);-` | 「Multiples: Format as 0.0x」 |
| `text` | `@` | 「Years: Format as text strings」 |
| `date` | `yyyy-mm-dd` | 日期 |
| `datetime` | `yyyy-mm-dd hh:mm` | 日期时间 |
| `general` | `General` | 通用 |

- `preset` 与 `fmt` 互斥，同时出现报错。
- 需要表里没有的格式时用 `fmt` 写字面格式串，脚本原样透传。
- 单位写在 `columns` 表头里（如「收入（万元）」），这是约定不是格式。

---

## 6. 语义角色 `role`

把 xlsx SKILL.md 的 Color Coding Standards 固化，AI 写语义、脚本上色。

| `role` | 应用效果 | 对应 xlsx 技能条款 |
|---|---|---|
| `input` | 蓝色字体 `0000FF` | Blue: 硬编码输入、用户会改的场景值 |
| `formula` | 黑色字体 `000000` | Black: 所有公式与计算 |
| `link` | 绿色字体 `008000` | Green: 同工作簿跨表引用 |
| `external` | 红色字体 `FF0000` | Red: 外部文件链接 |
| `assumption` | 黄色底色 `FFFF00` | Yellow bg: 需关注的关键假设 |

- `role` 可为字符串或数组；`assumption` 作用于底色，可与字体色叠加。
- **自动推断**：顶层 `conventions: "financial-model"` 时，未显式写 `role` 的单元格
  按规则推断：有 `f` 且引用 `[n]...` → `external`；有 `f` 且引用工作簿内其他表 →
  `link`；有 `f` 其他 → `formula`；无 `f` 且为数字 → `input`。
- 不设 `conventions` 时**不自动上色**，保持纯数据表外观。

---

## 7. 硬约束（脚本强校验，违反即报错）

1. **列数一致**：`header=true` 时，每个 `rows[i]` 长度必须等于 `columns.length`。
   默认严格报错，信息含 sheet 名、行号、期望列数与实际列数。`--pad` 可改为补 `null`。
2. **编码类必须写成字符串**：手机号、订单号、商品编码、带前导零的编号、身份证、
   银行卡号等。写数字会丢前导零 / 超 15 位丢精度。
3. **有效数字 > 15 位的数值必须写成字符串**。
4. **空单元格写 `null`，不要写 `""`**（`ISBLANK()` / `COUNTBLANK()` 行为不同）。
   ⚠️ **实测限制**：openpyxl 写入 `""` 时产出 `<c r=".." t="inlineStr" />`（无内容），
   回读恒为 `null`——**本工具无法交付「空字符串」与「空单元格」的区别**。
   脚本对 `""` 报 `EMPTY_STRING` warning 提示该行为。
5. **公式必须用 `{"f": ...}`**，不允许 `"=..."` 字符串。
6. **`name` 唯一**，≤31 字符，不含 `[ ] : * ? / \`。
7. **`rows` 中不得重复出现表头行**。
8. **单元格换行**用 `\n`，脚本自动开启自动换行；`--no-wrap` 关闭。
9. **`preset` 与 `fmt` 互斥**。
10. **`f` 引用的工作表必须存在**（§4.2 规则 2）。

---

## 8. 完整示例

```json
{
  "$schema": "sheet-json/v1",
  "conventions": "financial-model",
  "sheets": [
    {
      "name": "估时汇总",
      "header": true,
      "columns": ["业务流", "页面数", "小计"],
      "rows": [
        ["C端 · 会员等级与成长值", 3, "1.5 天"],
        ["C端 · 充值参与", 3, "1.5 天"],
        ["C端小计", { "f": "SUM(B2:B3)" }, { "f": "SUM(C2:C3)" }],
        ["后台小计", 10, "5 天"],
        ["合计", { "f": "B4+B5" },
                 { "v": 9, "preset": "decimal", "role": "assumption",
                   "note": "口径依据：积分商城拆解 V2.2" }],
        ["人均小计", { "f": "B6/B4", "preset": "decimal" },
                     { "f": "C6/C4", "preset": "decimal" }]
      ],
      "columnWidths": { "A": 28, "B": 10, "C": 12 },
      "freeze": "A2"
    },
    {
      "name": "折算口径",
      "header": true,
      "columns": ["项目", "口径", "适用日期"],
      "rows": [
        ["锚点", "兑换确认 = 1 天", { "v": "2026-09-09", "t": "date" }],
        ["后台", "统一 0.5 天/页", { "v": "2026-09-09", "t": "date" }],
        ["合计校准", { "f": "'估时汇总'!C5" }, null]
      ]
    }
  ]
}
```
