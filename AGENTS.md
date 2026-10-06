# openOMSI Development Rules

本文件適用於本 repository 的所有開發工作。若本文件與使用者對目前任務的明確指示衝突，依使用者指示辦理。

## 開發前同步與分支

在開始任何新功能或修復前，必須先確認工作樹狀態，並同步 `upstream` 的目標分支：

```powershell
git status --short --branch
git fetch upstream
git switch main
git pull --ff-only upstream main
```

若目前有未提交變更，必須先保留或處理這些變更，再進行同步；不得覆蓋或捨棄既有工作。

每個新功能或修復都必須從已同步的 `main` 建立獨立分支。分支名稱使用 `codex/` 前綴，並以簡短英文描述目的，例如：

```powershell
git switch -c codex/generic-addon-loading
```

不得直接在 `main` 上開發或提交變更。

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

## 實作、測試與交付

- 先閱讀相關模組、呼叫端與現有測試，再設計通用介面與測試案例。
- 變更完成後執行與範圍相符的驗證；至少遵循 `CONTRIBUTING.md` 要求的 `cargo test --workspace` 與 `cargo build --release`，必要時執行 `omsi-check`。
- 使用者要求測試 executable 時，必須先完成建置並提供可執行檔或其所在位置，讓使用者自行測試。
- 在使用者確認測試結果前，不得自行建立或提交 PR。使用者決定提 PR 後，才依專案要求建立英文 PR，並確認 base repository、base branch、commit 與 changed files 僅包含本次工作。

## Pull Request

PR 必須遵循 [`CONTRIBUTING.md`](CONTRIBUTING.md) 及 repository 的 PR template：

- PR title 與 body 一律使用英文。
- 說明使用者可觀察到的行為、通用架構設計、測試方式與結果。
- 維持單一變更主題，不混入無關修改。
- 附上必要的測試或建置證據；不在 repository 提交 build output、cache 或大型產物。
