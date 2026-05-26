---
name: er-bundle
description: "Use this skill when the user wants to turn SQL DDL or design Markdown into a long-lived ER design bundle (.erd.json) that drives an interactive web page — not a one-shot Mermaid snippet. Trigger when the request mentions an ER / entity-relationship diagram together with color-coded layers / sub-domains, multiple diagram views of the same schema, a write-flow / data-flow narrative, or an old-vs-new design decision log. Also trigger when the user asks to update an existing .erd.json or anything matching erd-bundle.schema.json. Do NOT trigger for pure 'draw a quick ER diagram' requests — Mermaid erDiagram is faster for that."
---

# er-bundle — ER 設計 bundle 產生流程

## 你要產出的東西

**主成品**:一份符合 `references/schema.json` 的 JSON(慣例副檔名 `.erd.json`)。

**選用成品**(使用者要求時):把 bundle 灌進宿主 HTML,產生可互動頁面。最小可用模板見 `examples/demo.html`(在這份 skill 之外的 repo 根目錄;路徑由使用者提供)。

## 為什麼用 bundle 不用 Mermaid

| 需求 | bundle 支援 | Mermaid `erDiagram` |
|---|---|---|
| 顏色分區(layers) | ✅ | ❌ |
| 多視角(同 schema 多張圖,不同子集) | ✅ `diagrams[]` | 需手動拆多個 code block |
| 流程卡 / 寫入路徑 | ✅ `dataFlows` | ❌ |
| 設計決策對照(old vs new) | ✅ `designDecisions` | ❌ |
| 自訂座標 + localStorage 記住拖曳 | ✅ `positions` | ❌ |
| GitHub 原生渲染 | ❌ 需宿主頁 | ✅ |

**判斷**:輸出靜態圖、放 README → 用 Mermaid;產品設計文件、會持續演化、要給多角色看 → 用本 skill。

## 工作流程

1. **讀 schema**:`references/schema.json` 確認必填欄位。**先看一次再開工**,新版有 `tableConstraints` / `onDelete` / `cardinality` / `enumValues` 等欄位容易漏掉。
2. **解析輸入**:從 SQL DDL 或 Markdown 抽出
   - 表名、欄位、型別
   - 單欄 PK/FK/UQ → `cols[*].tag`
   - 複合 PK/UQ、INDEX、CHECK → `tableConstraints`
   - `NOT NULL` → `nullable: false`、`DEFAULT` → `default`(原樣字串)
   - FK 行為 `ON DELETE/UPDATE` → 對應欄位
   - 列舉(從 CHECK 或 comment 判斷) → `enumValues`
3. **分區與狀態**:依語意填 `layers`,`tables[*].layer` / `status`(`existing` / `new` / `future`) / `comment`。
4. **每張圖**:`diagrams[]` 至少一張「全圖」;表多時補子網域圖。`connections.label` 通常是 FK 欄位名,並補 `cardinality`:
   - FK 欄位 **NOT NULL** → `"1:N"`(或 `"1:1"` 若對方該欄又是 UNIQUE)
   - FK 欄位 **NULL 允許** → `"0:N"`(或 `"0:1"`),代表「可選的上游」
   - 多對多中介表 → 中介表分別跟兩端各畫 `"1:N"` 即可
   - 範例見 [examples/team.erd.json](examples/team.erd.json):users / teams 互相可選引用,自然展示 `0:1` 與 `0:N`
   - **`dashed: true`** 只用於「邏輯關聯,非實體 FK」(例如反向自參考、跨服務邏輯引用、暫未實作但語意存在的關係)。一般 FK **不要** 設 dashed,渲染預設就是實線。
   - **`isNew: true`** 只用於「diff view」(這份 bundle 對比 v 舊版,標出新增的關係)。一般首次產出 **不要** 對所有連線設 isNew,會失去突顯效果。
5. **座標**:見 `references/layout-heuristics.md`。
6. **(選用)** 寫 `dataFlows`(寫入路徑、actor → action)與 `designDecisions`(old vs v3)。範例見 `examples/ecommerce.erd.json`。
7. **驗證**:`python3 scripts/validate.py <你的-bundle>.json`。**必跑**,沒過不交件。

## 反例(常見錯誤)

- ❌ `connections` 裡的 `from` / `to` 寫了不在 `tables` 裡的表名 → schema 不會擋,但宿主頁會壞。**自己 cross-check**。
- ❌ `positions` 漏掉某張表 → 那張表不會出現在圖上。每張 `diagram` 的 `positions` keys 必須涵蓋該圖要顯示的所有表。
- ❌ `tables[*].layer` 指到不存在的 layer key → schema 不會擋,宿主頁找不到顏色就崩。
- ❌ 複合 PK 把每一欄都標 `tag: "PK"` → 語意錯。複合鍵**只能**放 `tableConstraints`。
- ❌ `default: 'active'`(JS 字面意義)寫成 `default: active` → JSON 不合法或語意錯。`default` 是字串,原樣保留 SQL 字面(含引號或函式名)。
- ❌ 把每條連線都設 `dashed: true` 或 `isNew: true` → 失去語意。dashed = 邏輯關聯;isNew = diff view 才用。一般 FK 兩者皆不設。

## 參考檔案

- `references/schema.json` — 權威 JSON Schema
- `references/layout-heuristics.md` — 座標決定法則
- `examples/minimal.erd.json` — 最小範例(兩表一線)
- `examples/ecommerce.erd.json` — 完整範例(7 表、2 視角、含 dataFlows + designDecisions)
- `examples/ecommerce.sql` — 對應的原始 DDL
- `examples/team.erd.json` — 0:1 / 0:N 範例(可選關聯)
- `examples/team.sql` — 對應的原始 DDL
- `scripts/validate.py` — schema 驗證
