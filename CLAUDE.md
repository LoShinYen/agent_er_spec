# CLAUDE.md

本 repo 定義「ER 設計 bundle」(`.erd.json`)的規格與一份可載入此 bundle 的最小互動頁。把 SQL DDL / Markdown 設計稿轉成 bundle 是核心工作流。

## 在這個 repo 該怎麼做事

- **要把 SQL/MD 轉成 bundle** → 走 [plugins/er-bundle/skills/er-bundle/SKILL.md](plugins/er-bundle/skills/er-bundle/SKILL.md) 的工作流。SKILL.md 是產出步驟、cardinality 規則、反例的權威來源。
- **要看規格** → [erd-bundle.schema.json](erd-bundle.schema.json) 是權威 schema。`plugins/er-bundle/skills/er-bundle/references/schema.json` 是同檔的副本(CI 會 diff 防漂移)。
- **要把 bundle 渲染成可開的 HTML** → `python3 plugins/er-bundle/skills/er-bundle/scripts/render_html.py <bundle.json> -o out.html`。
- **要驗證 bundle** → `python3 plugins/er-bundle/skills/er-bundle/scripts/validate.py <bundle.json>`(schema + cross-check)。

## 改規格的硬性限制

當編輯 schema 或範例時要守的不變式:

- `diagrams[n].positions` 出現的每個表名,必須在 `tables` 有定義。
- `tables[*].layer` 必須是 `layers` 的其中一個 key。
- `connections[*].from` / `to` 必須出現在該 diagram 的 `positions` 裡。
- 任何 schema 變更要保持 backward compatible(新欄位都是 `optional`),否則既有 bundle 會破。
- 改 `erd-bundle.schema.json` 後,必須同步 `plugins/er-bundle/skills/er-bundle/references/schema.json`(`cp` 即可);CI 有 diff 步驟會擋。

## Bundle → 宿主 JS 常數對照

當把 bundle 灌進新的宿主頁(非 demo.html)時,以下是常見的命名對應:

| Bundle 路徑 | 宿主慣例變數名 |
|-------------|----------------|
| `layers` | `LAYERS` |
| `tables` | `TABLES` |
| `constants.cardW` / `cardHApprox` | `CARD_W` / `CARD_H_APPROX`(預設 `214` / `172`) |
| `diagrams[n].canvas` | 例如 `MY_CANVAS` |
| `diagrams[n].positions` | 例如 `MY_POS` |
| `diagrams[n].connections` | 例如 `MY_CONNS` |
| `diagrams[n].id` | 傳入 `ERDiagram` 的 `diagId`(`localStorage` 鍵前綴) |
| `dataFlows` / `designDecisions` | 例如 `DATA_FLOWS` / `DESIGN_DECISIONS`(選填) |

宿主頁只負責拖曳、縮放、連線繪製、modal — 不負責 schema 驗證或座標計算。

## 測試

```bash
python3 -m unittest discover -s plugins/er-bundle/skills/er-bundle/tests
```

CI 在 push / PR 自動跑(`.github/workflows/ci.yml`)。

## 不要做的事

- 不要在 `CLAUDE.md` 或 `SKILL.md` 裡 duplicate 對方的內容 — 一個是 repo 概覽,一個是產出工作流。
- 不要直接修改 `.claude/skills/er-bundle/` — 那是 symlink,改 `plugins/er-bundle/skills/er-bundle/` 就好。
- 不要把測試用的暫時 bundle(如 `/tmp/test_*.erd.json`)commit 進 repo。
