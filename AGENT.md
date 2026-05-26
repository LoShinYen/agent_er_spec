# Agent 操作說明：從規格到 ER 互動頁

本檔與同目錄 `erd-bundle.schema.json` 為一組：**不要依賴本目錄外的路徑或檔名**；宿主專案可自行命名 HTML/JS。

## 你應該產出的成品

1. **主要**：一個符合 `erd-bundle.schema.json` 的 JSON 檔（慣例副檔名 `.erd.json`）。
2. **宿主頁**：在任意 HTML 中載入 React（或你選用的框架），並提供下列 **JavaScript 全域或模組匯出**（名稱可依宿主慣例，但語意需對齊）：
   - `LAYERS` ← `bundle.layers`
   - `TABLES` ← `bundle.tables`
   - 每張圖：`canvas`、`positions`、`connections` ← `bundle.diagrams[i]` 的對應欄位
   - `ERDiagram` 的 `diagId` ← `bundle.diagrams[i].id`（用於 `localStorage` 鍵前綴）
   - 卡片幾何常數：`CARD_W`、`CARD_H_APPROX` ← `bundle.constants` 或預設 `214` / `172`

參考行為與 DOM/CSS 結構：開啟 `examples/demo.html` 檢視（可複製其 `ERDiagram`、`TableCard`、modal 邏輯）。

## 建議工作流程

1. 讀 `erd-bundle.schema.json` 確認必填欄位。
2. 讀使用者提供的 **SQL DDL** 或 **Markdown 表格**，抽出表名、欄位、PK/FK/UNIQUE、`NOT NULL`、`DEFAULT`、FK 行為(`ON DELETE` / `ON UPDATE`)、`CHECK`、`INDEX`、複合鍵。
3. 填寫 `layers`（分區顏色）、`tables[].layer` / `status` / `comment`。
4. 欄位層級填 `nullable` / `default` / `onDelete` / `onUpdate` / `enumValues`;複合 PK/UQ、INDEX、CHECK 放 `tables[*].tableConstraints`。
5. 為每個視角建立 `diagrams[]`：`positions`（表中心座標）、`connections`（`from` → `to`，`label` 多為 FK 欄位名,建議補 `cardinality`)。FK 可空時用 `0:N` / `0:1`,NOT NULL 時用 `1:N` / `1:1`。
6. 需要流程卡或決策對照時，再填 `dataFlows`、`designDecisions`（宿主頁需自行渲染這兩段資料）。

完整範例見 `examples/ecommerce.erd.json`(對應 `sources/ecommerce.sql`)。

## JSON → 宿主常數對照

| Bundle 路徑 | 宿主慣例變數名 |
|--------------|----------------|
| `layers` | `LAYERS` |
| `tables` | `TABLES` |
| `diagrams[n].canvas` | 例如 `MY_CANVAS` |
| `diagrams[n].positions` | 例如 `MY_POS` |
| `diagrams[n].connections` | 例如 `MY_CONNS` |
| `diagrams[n].id` | 傳入 `ERDiagram` 的 `diagId` |
| `dataFlows` | 例如 `DATA_FLOWS`（選填） |
| `designDecisions` | 例如 `DESIGN_DECISIONS`（選填；單筆物件常含 `old` 與 `v3` 兩段文字） |

## 注意

- 出現在 `diagrams[n].positions` 的每個表名，都必須在 `tables` 有定義。
- `tables[*].layer` 必須是 `layers` 的其中一個 key。
- 連線的幾何端點由 UI 依卡片中心與 `CARD_W` / `CARD_H_APPROX` 計算；agent 只維護座標與連線列表即可。
