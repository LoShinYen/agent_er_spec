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

## 改動時的同步清單

過去多次發生「改了行為但文件留在舊狀態」、「commit 前漏 stage 檔案」、「改名沒掃乾淨引用」。任何 PR / commit 前,**對著下面這份清單跑一遍**:

**改 SKILL.md 的觸發描述或工作流** →
- [ ] `README.md` 與 `README.zh-TW.md` 的 Skill 段落是否仍正確描述行為?
- [ ] 例:把 HTML 從「選用」改為「預設產出」時,兩份 README 都要同步。

**改 `erd-bundle.schema.json`** →
- [ ] `cp` 到 `plugins/er-bundle/skills/er-bundle/references/schema.json`(CI 會擋,但本機可先驗)
- [ ] 是否需要新範例展示新欄位?(像 `0:N` 對應 `examples/team.erd.json`)
- [ ] `README.md` / `README.zh-TW.md` 的「規格重點」段落?

**改名 / 移動檔案** →
- [ ] `grep -rn '<舊名>' --include='*.md' --include='*.yml' --include='*.json' .` 把所有引用掃出來修
- [ ] 包含 CI workflow paths、scripts 內部相對路徑

**改 `.claude-plugin/marketplace.json` 或 `plugin.json`** →
- [ ] 對齊 `claude-plugins-official` 的格式約定(`source` 用 `./plugins/<name>` 字串 或 git-subdir 物件)
- [ ] Plugin install 失敗時去翻 `~/Library/Logs/Claude/main.log`,不要憑空猜
- [ ] 改了 `version` 後通知測試者:**移除整個 marketplace → 重新 Add**,不是「uninstall + reinstall plugin」。Claude Code 對 custom marketplace **不會自動 `git pull`**,reinstall 只是讀已有 cache。手動觸發 fetch 的方法:刪掉 marketplace 重 Add,或在 `~/.claude/plugins/marketplaces/<name>/` 內 `git fetch && git reset --hard origin/main`。
- [ ] Install cache 路徑是 `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`,以 version 為 key。Bump version 才會建立新 cache;不 bump 改 SKILL.md / scripts 不會被 reinstall 看到。

**Commit 前固定動作** →
- [ ] `git status` 確認所有相關檔案都已 stage(不是只看「我有改」就以為進去了)
- [ ] 跑 `python3 -m unittest discover -s plugins/er-bundle/skills/er-bundle/tests` 過再 commit

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
