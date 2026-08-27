---
outline: deep
---

# 兼容性與限制

## 系統要求

| 要求 | 說明 |
|------|------|
| 最低系統版本 | macOS 12.0 (Monterey) |
| 架構 | Intel x86_64 / Apple Silicon (arm64) |
| 權限 | 完全磁盤訪問權限 |
| 外部存儲 | 需要至少一個外部存儲設備 |

## 功能兼容性

### 按 macOS 版本

| 功能 | macOS 12.0 - 15.0 | macOS 15.1+ |
|------|:---:|:---:|
| 應用遷移（Stub Portal） | ✓ | ✓ |
| 數據目錄遷移 | ✓ | ✓ |
| 目錄遷移（自定義文件夾） | ✓ | ✓ |
| 代碼簽名管理 | ✓ | ✓ |
| App Store 應用遷移到外置硬盤 | ✗ | ✓ |
| App Store 應用在外置硬盤原地更新 | ✗ | ✓ |
| iOS 應用遷移 | ✓ | ✓ |

::: warning ⚠️ macOS 15.1 以下的 App Store 應用
macOS 15.1（Sequoia）之前的系統不支持 App Store 應用安裝到外置硬盤。需在 AppPorts 設置中手動開啓「App Store 應用遷移」開關，且應用更新需手動二次遷移以覆蓋。
:::

### 按應用類型

| 應用類型 | 遷移 | 還原 | 自動更新 | 說明 |
|----------|:---:|:---:|:---:|------|
| 原生 macOS 應用 | ✓ | ✓ | ✓ | 最佳兼容性 |
| Sparkle 應用 | ✓ | ✓ | 需鎖定 | 鎖定後阻止應用內更新，需遷回後更新 |
| Electron 應用 | ✓ | ✓ | 需鎖定 | 同 Sparkle |
| Chrome / Edge（自定義更新器） | ✓ | ✓ | ✓ | 更新程序安裝到本地，不破壞外部副本 |
| App Store 應用（macOS 15.1+） | ✓ | ✓ | ✓ | 原生外置安裝，App Store 可直接更新 |
| App Store 應用（macOS <15.1） | ✓ | ✓ | 需手動 | 更新需二次遷移 |
| iOS 應用（Mac 版） | ✓ | ✓ | ✓ | 使用 iOS Stub Portal |
| 系統應用 | ✗ | — | — | SIP 保護，不可遷移 |

::: warning ⚠️ 受保護應用遷移
App Store 應用或 root 所有的應用可能因 macOS 權限無法由 AppPorts 自動刪除或替換本地副本。出現受保護應用提示時，建議先在 Finder 中手動移動到外部存儲，再回到 AppPorts 創建本地鏈接。
:::

### 按數據目錄類型

| 數據目錄類型 | 遷移 | 風險 |
|-------------|:---:|------|
| `~/Library/Application Support/` | ✓ | 中 — 可能使用文件鎖或 SQLite WAL 日誌 |
| `~/Library/Preferences/` | ✓ | 低-中 — `cfprefsd` 緩存可能導致過期讀取 |
| `~/Library/Containers/` | ✓ | 中 — 同一 Team 下多應用共享 |
| `~/Library/Group Containers/` | ✓ | 中 — 共享數據可能干擾其他應用 |
| `~/Library/Caches/` | ✓ | 低 — 緩存可重建 |
| `~/Library/Logs/` | ✓ | 低 — 僅日誌文件 |
| `~/Library/WebKit/` | ✓ | 中 — WebKit 本地存儲 |
| `~/Library/HTTPStorages/` | ✓ | 低 — 網絡會話存儲 |
| `~/Library/Application Scripts/` | ✓ | 低 — 擴展腳本 |
| `~/Library/Saved Application State/` | ✓ | 低 — 窗口狀態恢復 |
| `~/.npm`、`~/.m2` 等 dot-folder | ✓ | 低 — 開發工具緩存 |
| 用戶主目錄下的自定義文件夾 | ✓ | 視內容而定 — 遷移前應關閉正在寫入的應用或工具 |

::: warning ⚠️ 自定義目錄範圍
目錄遷移面向用戶主目錄中的真實文件夾；不能選擇文件、符號鏈接、外部目標內部路徑、系統目錄，或與已管理項目互相包含的路徑。
:::

## 不可遷移的內容

### 受 SIP 保護

| 路徑 | 原因 |
|------|------|
| macOS 系統應用（Safari、Finder 等） | 系統完整性保護 |
| `~/Library/Containers/` 頂層目錄 | macOS 系統保護 |

### 包含路徑引用

| 路徑 | 原因 |
|------|------|
| `~/.local` | 包含可執行文件路徑引用，遷移後命令行工具可能失效 |
| `~/.config` | 包含絕對路徑配置，遷移後工具配置可能失效 |

## 外部存儲要求

| 要求 | 說明 |
|------|------|
| 文件系統 | 支持 APFS、HFS+、exFAT |
| 最小空間 | 視遷移應用大小而定 |
| 接口 | USB、Thunderbolt、NVMe 均支持 |
| 保持連接 | 遷移後外部存儲需保持連接，否則應用無法啓動 |

::: tip 💡 文件系統建議
- **APFS**：推薦，支持克隆、快照，性能最佳
- **HFS+**：兼容性好，適合舊款 Mac
- **exFAT**：跨平臺兼容，但不支持硬鏈接和克隆
:::
