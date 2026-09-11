---
name: sheet-json
description: "文本和Excel转换工具。当需要读取 .xlsx 内容；把表格内容写成 .xlsx；内容转化成excel；excel内容读取时使用该技能"
---

# sheet-json：文本 和 excel 内容转换工具

## 环境

脚本需要 openpyxl，已装在项目 `.venv`：

```bash
.venv/bin/python scripts/sheetjson.py --help
```

若缺依赖：`uv pip install --python .venv/bin/python openpyxl`


## 工作流

### 文本 转 .xlsx
1. 文本内容按照 [format-spec](./reference/format-spec.md) 输出成 json
2. 使用 [sheet-json](./scripts/sheetjson.py) 转换成 .xlsx

### .xlsx 转 文本
1. 使用 [sheet-json](./scripts/sheetjson.py) 转换成 json 文本
2. 按照用户要求，将 json 文本，转换成目标格式


**禁止直接读写 xlsx 二进制**，只产出 JSON，其余交给脚本。

## 脚本使用

```bash

sheetjson.py to-json  报表.xlsx -o 报表.json     # 1. 读出来
#  2. AI 编辑 报表.json
sheetjson.py check    报表.json                  # 3. 校验（exit 1 = 有错）
sheetjson.py to-excel 报表.json -o 报表_v2.xlsx  # 4. 写回去

# 如果有 .venv 环境，脚本需要使用 .venv 环境运行
.venv/bin/python sheetjson.py to-json 报表.xlsx -o 报表.json
.venv/bin/python sheetjson.py check 报表.json
.venv/bin/python sheetjson.py to-excel 报表.json -o 报表_v2.xlsx
```

`to-excel` 会**先校验再写**：有 error 就拒绝生成文件。`--force` 可强制写出。

## 命令参考

| 命令 | 作用 |
|---|---|
| `to-json <xlsx> [-o out.json]` | 读出。`--sheet 名称`（可重复）、`--no-cached`、`--no-header`、`--no-fmt`、`--no-role`、`--keep-empty` |
| `to-excel <json> [-o out.xlsx]` | 写入。`--pad`（列数不足补 null）、`--no-wrap`、`--strict`、`--force`、`--extra-funcs` |
| `check <json\|xlsx>` | 静态校验，输出 JSON 报告到 stdout。`--strict`、`--extra-funcs` |
| `verify <xlsx>` | 往返比对。`--tolerance`、`--sheet`、`--keep-empty` |

退出码：`0` 通过/仅有 warning，`1` 有 error 或往返有差异，`2` 参数/IO 错误。

## 必须遵守的硬约束

违反即报 error：

1. **列数一致**：`header=true` 时每个数据行长度必须等于 `columns` 长度。
2. **编码类写成字符串**：手机号、订单号、商品编码、带前导零的编号、身份证。
   写数字会丢前导零；**超过 15 位有效数字**一律用字符串。
3. **空单元格写 `null`**，不要写 `""`。
4. **公式用 `{"f": ...}`**，不要写 `"=SUM(...)"` 字符串——
   文本单元格本来就可能以 `=` 开头，字符串约定会产生歧义。
5. **日期必须显式 `t`**：`"2026-09-09"` 直接写进 Excel 是**文本**不是日期。
6. **`preset` 与 `fmt` 互斥**。
7. 工作表名唯一、≤31 字符、不含 `[ ] : * ? / \`。
8. `f` 里引用的工作表必须存在。



## 校验报告与修复循环

```bash
$PY $S check 报表.json
```

```json
{
  "status": "errors_found",
  "errors": [
    { "sheet": "估时汇总", "cell": "C5", "rule": "REF_SHEET",
      "message": "引用了不存在的工作表「明细」", "formula": "SUM(明细!A1:A5)" }
  ],
  "warnings": [
    { "sheet": "估时汇总", "cell": "C7", "rule": "MAGIC_NUMBER",
      "message": "公式含魔数 1.05，建议改用单元格引用", "formula": "B5*1.05" }
  ],
  "stats": { "sheets": 2, "formulas": 42, "cached_errors": 0, "unverified": 42 }
}
```

**修复循环**：`check` → 按 `errors[].sheet/cell/rule` 定点改 JSON → 重跑，
直到 `status` 不再是 `errors_found` → `to-excel`。

**能查的错误**：括号/引号不配对、引用不存在的工作表、坐标越界、公式含错误字面量、
未知函数名（`#NAME?`）、循环引用、空公式；xlsx 输入还会扫已保存的缓存错误值。

## 能力边界

本工具**没有公式计算引擎**，这些做不到，不要假装做到了：

1. **不计算公式真值**。`{"f":"SUM(B2:B3)"}` 旁的 `v` 是**期望值提示**，
   openpyxl 不会把它写进文件（只写出空的 `<v/>`）。Excel 打开时会自动重算。
2. **动态运行期错误无法预判**：分母来自单元格且某行为 0 导致的 `#DIV/0!`、
   查找未命中的 `#N/A`，静态查不出来。报告的 `unverified` 字段会明确告诉你
   **有多少公式的真值未被验证**——这个数字不是 0 时不要声称"公式没问题"。
3. **`""` 与 `null` 无法区分**：openpyxl 写 `""` 产出 `<c t="inlineStr" />`，
   回读恒为 `null`。脚本报 `EMPTY_STRING` warning。
4. **不保留样式**：只支持 `role` 的语义化配色，不支持任意 RGB、字号、边框、
   图表、图片、条件格式、数据验证、透视表。整表重写会丢原有格式。

> 若需要「改已有 Excel 且保住原有格式与公式」，不要用整表重写——
> 应改用补丁思路（只改指定单元格后原位保存），本格式不覆盖该场景。


## 参考文档

- `reference/format-spec.md` — 完整格式规范（设计取舍、能力边界、逐条规则）
- `sheet.schema.json` — 机器可校验的 JSON Schema (draft-07)

