# CLAUDE.md（繁體中文版）

本文件為 Claude Code (claude.ai/code) 在此專案中工作時的操作指南。

> 英文原文：[/CLAUDE.md](../../CLAUDE.md)

## 快速開始

```bash
uv sync --all-extras            # 安裝所有依賴套件
uv run pre-commit install       # 安裝 pre-commit hooks
cp .env.example .env            # 設定 API 金鑰（編輯 .env 填入 ANTHROPIC_API_KEY）
```

## 開發指令

```bash
make format        # 用 ruff 格式化程式碼
make lint          # 執行 lint 檢查
make check         # 執行格式檢查 + lint（提交前請先執行）
make fix           # 自動修復問題 + 格式化
make test          # 執行 pytest 測試
```

### Notebook 測試

```bash
# 結構測試（快速，不需 API 呼叫）
make test-notebooks                                        # 所有 notebook
make test-notebooks NOTEBOOK=tool_use/calculator_tool.ipynb  # 單一 notebook
make test-notebooks NOTEBOOK_DIR=capabilities              # 單一目錄

# 執行測試（較慢，需要 ANTHROPIC_API_KEY）
make test-notebooks-exec NOTEBOOK=tool_use/calculator_tool.ipynb
```

## 程式碼風格

- **每行長度：** 100 字元 | **引號：** 雙引號 | **格式化工具：** Ruff
- Notebook 有寬鬆的規則：允許檔案中間 import (E402)、重複定義 (F811)、變數命名 (N803, N806)

## Git 工作流程

**分支命名：** `<使用者名稱>/<功能描述>`

**Commit 格式（conventional commits）：**
```
feat(scope): add new feature
fix(scope): fix bug
docs(scope): update documentation
style: lint/format
```

## 重要規則

1. **API 金鑰：** 絕對不要提交 `.env` 檔案。一律使用 `os.environ.get("ANTHROPIC_API_KEY")`

2. **依賴管理：** 使用 `uv add <套件>` 或 `uv add --dev <套件>`。不要直接編輯 pyproject.toml。

3. **模型版本：** 使用當前的 Claude 模型，詳見 docs.anthropic.com。
   - Sonnet: `claude-sonnet-4-6`
   - Haiku: `claude-haiku-4-5`
   - Opus: `claude-opus-4-6`
   - **禁止使用帶日期的模型 ID**（例如 `claude-sonnet-4-6-20250514`），一律使用不帶日期的別名。
   - **Bedrock 模型 ID** 格式不同，請參考文件：
     - Opus 4.6: `anthropic.claude-opus-4-6-v1`
     - Sonnet 4.5: `anthropic.claude-sonnet-4-5-20250929-v1:0`
     - Haiku 4.5: `anthropic.claude-haiku-4-5-20251001-v1:0`
     - 建議加上 `global.` 前綴使用全域端點：`global.anthropic.claude-opus-4-6-v1`
     - 注意：Opus 4.6 之前的 Bedrock 模型需要帶日期的 ID。

4. **Notebook：**
   - 保留 notebook 中的輸出結果（這是刻意的，用於展示）
   - 一個 notebook 只涵蓋一個概念
   - 確保 notebook 能從頭到尾順利執行

5. **品質檢查：** 提交前執行 `make check`。

## Pre-commit Hooks

提交時自動執行：ruff 格式化、ruff lint（含自動修復）、notebook 結構驗證（`scripts/validate_notebooks.py`）、`authors.yaml` 排序檢查。

## CI 工作流程

所有工作流程皆為自動觸發，沒有僅能手動觸發的項目。

### Pull Request 觸發（全部 9 個工作流程）

| 工作流程 | 觸發路徑 | 備註 |
|---|---|---|
| **lint-format** | `.py`, `.ipynb`, `pyproject.toml`, `uv.lock`, `Makefile` | |
| **notebook-tests** | `.ipynb`, `tests/notebook_tests/**`, `pyproject.toml`, `uv.lock` | 執行測試僅限維護者（需要 API 金鑰） |
| **notebook-quality** | `.ipynb`, `pyproject.toml`, `uv.lock` | |
| **notebook-diff-comment** | `.ipynb` | |
| **claude-pr-review** | `.ipynb`, `.py`, `.github/workflows/**`, `pyproject.toml`, `uv.lock` | 也會在 `ready_for_review` 時觸發 |
| **claude-model-check** | `.ipynb`, `.py`, `.md` | |
| **claude-link-review** | `.md`, `.mdx`, `.ipynb`, `README.md` | |
| **links** | 所有檔案（無路徑過濾） | |
| **verify-authors** | `authors.yaml`, `registry.yaml` | |

### Push 到 main 觸發（4 個工作流程）

- **lint-format** — `.py`, `.ipynb`
- **notebook-tests** — `.ipynb`, `tests/notebook_tests/**`
- **notebook-quality** — `.ipynb`
- **verify-authors** — `authors.yaml`, `registry.yaml`

### 排程觸發

- **links** — 每週日 UTC 00:00 執行完整連結驗證（`cron: "0 0 * * SUN"`）

### 手動觸發（`workflow_dispatch`）

以下工作流程除了自動觸發外，也支援從 GitHub Actions 頁面手動執行：
- **claude-link-review**、**claude-model-check**、**claude-pr-review** — 需輸入 PR 編號
- **links** — 無需參數

## Slash 指令

- `/notebook-review` — 檢閱 notebook 品質
- `/model-check` — 驗證 Claude 模型參考
- `/link-review` — 檢查變更檔案中的連結
- `/add-registry` — 新增 notebook 項目到 registry.yaml
- `/review-pr` — 檢閱開啟中的 Pull Request
- `/cookbook-audit` — 根據風格指南審核 notebook

## 新增 Cookbook

1. 在適當的目錄中建立 notebook
2. 在 `registry.yaml` 中新增項目（必填欄位：`title`、`path`、`categories`、`authors`、`date`）
3. 如果是新貢獻者，在 `authors.yaml` 中新增作者資訊（依字母順序排列）
4. 執行 `make check` 後提交 PR

### registry.yaml 項目格式

```yaml
- title: My Cookbook Title
  description: Brief description of what this cookbook demonstrates.
  path: capabilities/my_cookbook.ipynb
  authors:
  - github-username
  date: 'YYYY-MM-DD'
  categories:
  - Tools          # 可選值：Agent Patterns, Claude Agent SDK, Evals, Fine-Tuning,
                   #   Multimodal, Integrations, Observability, RAG & Retrieval,
                   #   Responses, Skills, Thinking, Tools
```
