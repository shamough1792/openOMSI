# openOMSI Development Rules

本文件適用於本 repository 的所有開發工作。若本文件與使用者對目前任務的明確指示衝突，依使用者指示辦理。

本檔在本機是 untracked、受 `.git/info/exclude` 保護，且不被任何開發分支追蹤，因此不會出現在 commit 或 PR diff 中；存放、版本化與還原方式見文末「規範檔的存放與保證」。

## 開發前同步與分支

在開始任何新功能或修復前，必須先確認工作樹狀態，並同步 `upstream` 的目標分支：

```powershell
git status --short --branch
git fetch upstream
git switch main
git merge --ff-only upstream/main
```

若目前有未提交變更，必須先保留或處理這些變更，再進行同步；不得覆蓋或捨棄既有工作。

若同步時發生 merge conflict，必須停止並回報衝突檔案與待決定的處理方式，不得猜測或覆蓋衝突內容。

每個新功能或修復都必須從已整合最新 `upstream/main` 的本地 `main` 建立獨立分支。不得直接從官方 `upstream/main` 建立，避免遺失本 repository 的開發規範。分支名稱不得使用 `codex/` 前綴，並使用簡短、清楚的英文描述目的，例如：

```powershell
git switch -c generic-addon-loading
```

不得直接在 `main` 上開發或提交變更。

## 建置產物與磁碟空間管理

- 開發前檢查建置目錄的空間占用，辨識本次工作可重用及已不再需要的產物，按需清理。
- 優先沿用既有建置目錄，避免為每個任務建立獨立 target 目錄，重複累積依賴產物。
- 任務結束後，清理本次建立且不再使用的臨時建置目錄。
- 保留日常開發可重用的編譯快取；空間不足時才清理 `target/debug` 或 `target/release`。
- 使用者尚在測試或尚未確認可移除的 executable，不得自行刪除。
- 清理範圍限於可重新產生的建置產物，不得刪除原始碼、資產、設定或 Git 資料。

## 核心架構原則

系統的核心邏輯與架構必須保持高度通用性（Generality）與獨立性（Abstraction）。所有新功能與修復都必須設計成可跨模組及 Add-on 一致套用的通用機制。

### 嚴禁

- 在 Core/Base 邏輯加入針對特定 Add-on 的條件分支，例如 `if (addonId == "bus_A")` 或 `if (isSpecialCase)`。
- 為單一 Add-on 的問題在公共模組加入強耦合補丁、繞過正常流程的 hack 或硬編碼例外。
- 把目前觀察到的單一資料或邊緣情境當成核心 API 的特殊契約。

### 必須

- 將差異化行為放在資料或設定檔、介面實作、事件鉤子（Event Hooks）或其他明確的 Extension Point。
- 將共通需求提煉為 Generic Engine/System，使未來 Add-on 能透過相同機制使用，不需修改 Core/Base 條件分支。
- 讓核心模組依抽象介面與穩定資料契約運作，避免依賴特定 Add-on 的名稱、路徑、資產或載入順序。
- 在變更說明中指出抽象邊界、適用範圍，以及新 Add-on 如何採用該機制。

若需求只能以單一 Add-on 特例完成，必須先重新設計為可泛化的能力；若確實無法泛化，應在實作前說明限制與架構取捨。

## 註解與文件語言

- 程式碼內所有註解必須使用英文，並只說明必要的原因、限制或不易從程式碼看出的設計意圖。
- 一般工作回報可使用繁體中文。
- Git commit subject/body、PR title/body、review comment 與 issue 內容必須使用英文，並遵循 repository 的既有要求。
- Commit 必須遵循 Conventional Commits，例如 `feat:`, `fix:`, `docs:`, `refactor:`，且每個 commit 只表達一個可獨立理解的目的。

## 實作、測試與交付

- 先閱讀相關模組、呼叫端與現有測試，再設計通用介面與測試案例。
- 變更完成後執行與範圍相符的驗證；至少遵循 `CONTRIBUTING.md` 要求的 `cargo test --workspace` 與 `cargo build --release`，必要時執行 `omsi-check`。
- 修改 Rust 原始碼後，提交前執行 `bash scripts/size-budget.sh`；若超出大小預算，優先拆分檔案或函式。只有在有明確理由時才調高 `scripts/size-budget.txt` 基準，並須在同一 PR 說明原因。
- 使用者要求測試 executable 時，必須先完成建置並提供可執行檔或其所在位置，讓使用者自行測試。
- 在使用者確認測試結果前，不得自行建立或提交 PR。使用者決定提 PR 後，才依專案要求建立英文 PR，並確認 base repository、base branch、commit 與 changed files 僅包含本次工作。
- 新功能分支的基準必須是已整合最新 `upstream/main` 的本地 `main`；建立 PR 前確認 base repository 為官方 repository，base branch 為 `main`。
- 禁止使用 `git push --force`。若確實需要重寫私有分支歷史，必須先取得明確授權，並使用 `--force-with-lease`。
- 修改 Core、載入器、格式解析或共用 API 時，必須驗證既有 Add-on 的相容性；無法執行完整相容性檢查時，必須在 PR 中說明原因。
- 新增依賴前必須確認其授權、維護狀態與跨平台支援；不得提交原始 OMSI 資產、下載內容、建置產物、cache 或個人設定。
- 測試 executable 應透過 PR comment、release artifact 或其他 repository 外部位置提供，不得提交至 repository；並記錄建置 commit、平台、架構與測試方式。

## Pull Request

PR 必須遵循 [`CONTRIBUTING.md`](CONTRIBUTING.md) 及 repository 的 PR template：

- PR title 與 body 一律使用英文。
- PR message 必須完整依照 [`.github/pull_request_template.md`](.github/pull_request_template.md) 的欄位與順序撰寫，不得省略或改用其他格式；未使用的欄位應明確填寫不適用原因。
- 說明使用者可觀察到的行為、通用架構設計、測試方式與結果。
- 維持單一變更主題，不混入無關修改。
- 每個 PR 只保留與該次修改直接相關的 commits；不得包含其他功能、修復或規範工作的 commits。
- 建立 PR 前必須檢查相對於 base branch 的 commit 清單與 changed files。若發現無關 commits，先從正確 base 重建或整理分支，再提交 PR。
- 若分支包含無關或過度零碎的 commits，必須在建立 PR 前整理；不得透過 PR 將其他功能、修復或規範變更一併帶入。
- 附上必要的測試或建置證據；不在 repository 提交 build output、cache 或大型產物。

## 規範檔的存放與保證

本規範同時滿足兩個條件：**每次開發都讀得到**，且**永不混入 commit 或 PR**。

內容來源（必須一字不差）：

1. 工作樹根目錄 `AGENTS.md`：untracked、受 `.git/info/exclude`（`/AGENTS.md`）保護，agent 每次開工時自動載入。
2. 版本化副本：`agents-rules` 分支的 `docs/DEVELOPMENT_RULES.md`。此分支是 **orphan 分支**（沒有父 commit、不基於 `main`），因此 push 它只會送出一個 commit、一個檔案，不會夾帶任何開發 commit；它也**永不 merge 進任何開發分支**，不會進到 PR diff。
3. 本機備份：`<git-common-dir>/openomsi-agent-rules.md`，位於共用 git 目錄內，`git clean -xdf`、checkout、worktree 操作都不會波及。

跨 agent 入口檔（同樣 untracked、同樣受 exclude 保護、同樣不得提交）：

- `AGENTS.md`：DSH、Codex CLI、Cursor 等直接讀取。
- `CLAUDE.md`：只含一行 `@AGENTS.md`，供 Claude Code 匯入（Claude Code 不原生讀取 `AGENTS.md`）。
- `GEMINI.md`：只含一行指向 `AGENTS.md` 的說明，供 Gemini CLI 等讀取。

必須遵守：

- 不得以 `git add`、`git commit` 或 PR 帶入 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`；不得在任何開發分支追蹤這三個檔名。
- 不得把 `agents-rules` 分支 merge 進開發分支，也不得把 `docs/DEVELOPMENT_RULES.md` 帶進開發分支。
- `agents-rules` 必須保持 orphan（`git rev-list --count agents-rules` 等於 1）；若它被重建成基於 `main` 的分支，push 會把未推送的開發 commit 一起送出。
- 修改規範時，同步更新上述所有位置，並重新套用 exclude 清單。

新機器或新 clone 還原（三段都要做，缺一就會失去「不混入」的保護）：

```powershell
git fetch origin agents-rules
git show agents-rules:docs/DEVELOPMENT_RULES.md | Set-Content AGENTS.md -Encoding utf8
Set-Content CLAUDE.md '@AGENTS.md' -Encoding utf8
Set-Content GEMINI.md 'Read AGENTS.md in the repository root and follow it.' -Encoding utf8
Add-Content .git/info/exclude "`n# fork-local development rules: on disk only, never committed`n/AGENTS.md`n/AGENTS.override.md`n/CLAUDE.md`n/GEMINI.md"
```

驗證方式：

- `git diff --name-only upstream/main...HEAD` 不得出現 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`。
- `git check-ignore -v AGENTS.md CLAUDE.md GEMINI.md` 三者都要命中 `.git/info/exclude`。
