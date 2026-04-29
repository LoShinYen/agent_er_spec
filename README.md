# Agent ER 規格（`agent_er_spec`）

本目錄可**單獨**放到 GitHub：內含 JSON Schema、給 agent 的說明、範例 bundle，以及可離線開啟的 **HTML 範例頁**，不依賴本目錄以外的檔案。

## 檔案一覽

| 路徑 | 用途 |
|------|------|
| `erd-bundle.schema.json` | **JSON Schema**：合法 bundle 的結構定義。 |
| `AGENT.md` | **給 agent 的流程**：從 MD / SQL → bundle JSON → 對應到宿主頁的 JavaScript 常數。 |
| `examples/minimal.erd.json` | 最小 bundle（兩表、一連線、一張圖）。 |
| `examples/demo.html` | 自包含互動頁：讀取頁內嵌的 bundle（與 `minimal.erd.json` 同步），示範拖曳、縮放、連線、點表 modal。 |
| `sources/` | 放置原始 DDL 或設計用 Markdown，供 agent 引用。 |

## 快速預覽

在檔案總管中直接開啟 `examples/demo.html`，或使用任意靜態伺服器於本目錄根路徑提供檔案即可。

若你修改 `minimal.erd.json`，請同步更新 `demo.html` 內 `<script type="application/json" id="er-bundle-json">` 的內容，兩者應保持一致。

## 與「完整產品頁」的關係

本目錄只定義 **資料形狀**（bundle）與 **最小可執行 UI**。完整應用可另外在宿主專案中加入分頁（架構說明、差異對照、資料流等）；那些 UI 不在此目錄範圍內，但選用欄位 `dataFlows`、`designDecisions` 已預留在 schema 中供擴充。
