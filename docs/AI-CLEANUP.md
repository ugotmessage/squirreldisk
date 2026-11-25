# AI 清理規格 v0.1（提案）
## 已確認架構
Cargo.toml: Rust 2021、eframe/egui 0.36、serde/serde_json、GUI 可選 ureq。src/tree.rs 使用 arena Tree/Node，含 allocated bytes、files、Kind、flags、source、version；小檔案聚合，未列全不可視為單檔。src/cli.rs 支援 scan --json --depth --top --fresh。src/ui/app.rs Session 含 SharedTree、selected、collector。src/safety.rs 已有 Ok/Caution/Forbidden 三級。src/update.rs 引用 crate::GITHUB_REPO。
## 整合
建議新增 src/ai/{mod,model,rules,provider,cache}.rs（實際命名可調整），以及 UI 分析模組。先從已選節點取得短時間唯讀快照，釋放鎖後再分析；以 source、tree version、節點路徑及本地身份綁定結果，過期標示。不要改動 scanner 或 headless agent 的依賴；網路功能限 GUI feature 或獨立 feature。
輸入：OS、來源類別、已去識別的相對路徑/類型、allocated bytes、files、掃描完整性；修改時間需另行唯讀取得，不假設 Tree 已提供。輸出 JSON：candidate_id、用途、可能軟體、風險(low/review/protected/unknown)、理由、證據、建議動作(none/review/trash_candidate)、不確定性。只接受現存候選 ID，拒絕任意路徑、指令及未知欄位值。模型輸出一律不可信，不是執行授權。
規則優先：已確認可再生快取、安裝檔、開發建置產物；辨識 Docker/VM/資料庫/備份/照片/雲端同步資料，預設保護或待確認。不能僅憑資料夾名稱判安全。AI 信心不降低 safety verdict。Hidden、Mount、Link、SmallFiles 聚合、未完整掃描、拒絕存取、Demo 禁止直接加入 AI 清理計畫。先本機；SSH/rclone 唯讀。
## 隱私與成本
本地規則無網路；Ollama 優先，遠端 provider 明確選用並預覽資料；預設不傳內容、完整私人路徑、host、帳號、token、憑證或環境變數。敏感名稱遮罩；API key 交由 OS 安全儲存，不寫 log/repo。批次、限額、超時、取消、模型/規則版本快取；錯誤顯示 unknown。檔名中的文字視為資料，不能成為指令。
## 清理（P2，非 MVP）
分析→預覽→勾選→確認→重新解析路徑/來源/身份與 safety 判定→原 collector/delete 執行→結果與錯誤紀錄。阻止 symlink/junction、路徑替換、跨 mount、祖先/子孫重複選取、過期結果及權限升級；執行層仍須驗證，無法安全驗證則停止。不得允許 AI shell 或直接永久刪除。專用工具如 Docker prune 僅顯示說明，未來以固定參數白名單另行審核。容量預估不等於實際回收，注意 hardlink、APFS clone、垃圾桶與快照。
## 驗收案例
無 AI 設定、provider timeout、惡意檔名、無效 JSON、敏感路徑遮罩、取消、掃描版本變動、Forbidden/Caution、連結替換、重複候選、遠端唯讀、掃描不完整；使用 fixture/tempdir。先 cargo fmt --check、cargo test、cargo test --no-default-features、cargo clippy（以實際環境可行性為準），再 GUI 手測。未執行不得標示通過。
