# Agent ER 規格(`agent_er_spec`)

[English](README.md) · **繁體中文**

![ci](https://github.com/LoShinYen/agent_er_spec/actions/workflows/ci.yml/badge.svg)

把 SQL DDL 或設計 Markdown,轉成一份**長期可維護**的 ER 設計 bundle(`.erd.json`),並由自包含 HTML 渲染成可拖曳、可縮放的互動頁。

跟 Mermaid `erDiagram` 的差異:本規格保留 **顏色分區(layers)、多視角(diagrams)、流程卡(dataFlows)、設計決策對照(designDecisions)** 等設計語意 — 這些是給團隊看的系統設計文件需要的,Mermaid 沒有。

## 兩種使用方式

### A. 當作規格 / 範例庫(原始用法)

直接讀根目錄檔案,自己手工或讓 agent 產出 bundle:

| 路徑 | 用途 |
|------|------|
| [erd-bundle.schema.json](erd-bundle.schema.json) | **JSON Schema**:bundle 的權威結構定義 |
| [CLAUDE.md](CLAUDE.md) | 給任何 Claude Code session 自動載入的 repo 指引(指向 SKILL.md 與 schema) |
| [examples/minimal.erd.json](examples/minimal.erd.json) | 最小範例:兩表一線 |
| [examples/ecommerce.erd.json](examples/ecommerce.erd.json) | 完整範例:7 表、2 視角、含 `dataFlows` + `designDecisions` |
| [examples/team.erd.json](examples/team.erd.json) | 可選關聯範例:`0:1` / `0:N` cardinality |
| [examples/demo.html](examples/demo.html) | 自包含互動頁,可離線直接開啟 |
| [sources/ecommerce.sql](sources/ecommerce.sql) · [sources/team.sql](sources/team.sql) | 範例對應的原始 DDL |

### B. 當作 Claude Code Skill(推薦)

[plugins/er-bundle/skills/er-bundle/](plugins/er-bundle/skills/er-bundle/) 是一份自包含的 Skill,描述、schema、範例、驗證腳本、渲染腳本一應俱全。Claude Code 會在使用者要產或更新 `.erd.json` 時自動觸發。

**Skill 觸發後預設會同時產出 `.erd.json` 與可互動 HTML 預覽**(自動開啟)。只想要 JSON 的話跟它說「只要 JSON」即可。下方 [工作流](#工作流) 的 CLI 指令是給「在 Claude Code 之外操作 bundle」或「手動編輯後重新渲染」用的。

安裝(三選一):

- **Plugin marketplace**:把 `.claude-plugin/marketplace.json` 加入 Claude Code 的 plugin marketplace。
- **User-wide**:複製 `plugins/er-bundle/skills/er-bundle/` 到 `~/.claude/skills/`,任何專案都能觸發。
- **Project-local**(開發 skill 本身時最方便):在 repo 根目錄跑
  ```bash
  mkdir -p .claude/skills && ln -sfn ../../plugins/er-bundle/skills/er-bundle .claude/skills/er-bundle
  ```
  `.claude/` 已在 `.gitignore` 內,symlink 不會被 commit;改 `plugins/er-bundle/skills/er-bundle/` 內容即時反映。

```
plugins/er-bundle/skills/er-bundle/
├── SKILL.md                  # 觸發描述 + 工作流 + 反例
├── references/
│   ├── schema.json
│   └── layout-heuristics.md  # 座標決定法則
├── examples/                 # minimal、ecommerce(JSON + SQL)、demo.html
└── scripts/
    ├── validate.py           # schema + cross-check
    └── render_html.py        # bundle → 可開的互動 HTML
```

## 規格重點

bundle 必填 `meta` / `layers` / `tables` / `diagrams`。完整欄位見 [schema](erd-bundle.schema.json),以下是常用但**易忽略**的選用欄位:

- **欄位層級**:`nullable`、`default`、`onDelete` / `onUpdate`、`enumValues`
- **表層級**:`tableConstraints[]` — 複合 PK/UQ、INDEX、CHECK(單欄請仍用 `cols[*].tag`)
- **連線**:`cardinality`(`1:1` / `1:N` / `N:N` / `0:1` / `0:N`)、`dashed`(邏輯關聯)。`0:*` 代表可選關聯,對應可空 FK
- **選用區塊**:`dataFlows`、`designDecisions`

## 工作流

```bash
# 1. 驗證(schema + cross-check FK 端點、layer 參照、positions 完整性)
python3 plugins/er-bundle/skills/er-bundle/scripts/validate.py examples/ecommerce.erd.json

# 2. 渲染成可開的 HTML
python3 plugins/er-bundle/skills/er-bundle/scripts/render_html.py examples/ecommerce.erd.json -o /tmp/out.html
open /tmp/out.html
```

兩個腳本都需要 `jsonschema`:`pip install --user jsonschema`。

## 與「完整產品頁」的關係

本目錄只定義 **資料形狀** 與 **最小可用 UI**。完整應用可在宿主專案另加分頁(架構說明、差異對照),`dataFlows` 與 `designDecisions` 已預留於 schema 中供擴充。
