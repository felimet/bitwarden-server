# 客戶端設定指南

Vaultwarden 完全相容官方 Bitwarden 客戶端。本文件說明如何在各平台的 Bitwarden 客戶端中指定自建伺服器地址。

## 支援平台一覽

| 平台 | 客戶端類型 | 下載來源 |
|------|-----------|----------|
| Chrome / Edge / Firefox / Safari | 瀏覽器擴充功能 | 各瀏覽器擴充商店 |
| Windows / macOS / Linux | 桌面應用程式 | [bitwarden.com/download](https://bitwarden.com/download/) |
| iOS | 行動 App | App Store |
| Android | 行動 App | Google Play |
| 終端機 | CLI | [bitwarden.com/download](https://bitwarden.com/download/) |

## 事前準備

1. 伺服器已依照 [部署步驟](deployment.md) 完成啟動，且可透過瀏覽器存取 Web Vault（例如 `https://vault.example.com`）。
2. 已在 Web Vault 上完成管理者帳號的註冊。
   - 帳號（Email）與主控密碼（Master Password）為所有客戶端的登入憑證。

---

## 瀏覽器擴充功能 / 桌面應用程式

1. 安裝並開啟 Bitwarden 應用程式或擴充功能。
2. **不要直接登入**。點選登入畫面的 **齒輪圖示** 或 **「Self-hosted / 自行代管」** 選項。
3. 在「伺服器 URL（Server URL）」欄位輸入自建伺服器地址：
   ```
   https://vault.example.com
   ```
4. 點選「儲存（Save）」返回登入畫面。
5. 輸入 Email 與 Master Password 完成登入。

> 設定完成後，所有密碼庫資料將與自建伺服器雙向同步，擁有完全自主的資料所有權。

## 行動應用程式（iOS / Android）

1. 在 App Store 或 Google Play 安裝 **Bitwarden 密碼管理員**。
2. 開啟 App。
3. 點選登入畫面上方的 **齒輪圖示**（右上角）。
4. 在「自行代管（Self-hosted）」區域填入伺服器 URL：
   ```
   https://vault.example.com
   ```
5. 點選「儲存」，返回登入畫面。
6. 輸入 Email 與 Master Password 登入。若已設定 2FA，此時將觸發雙重驗證流程。

### 建議啟用的安全功能

| 功能 | 說明 |
|------|------|
| **生物辨識解鎖** | 啟用 Face ID / Touch ID / 指紋辨識，免除每次輸入 Master Password |
| **PIN 碼解鎖** | 設定數字 PIN 作為快速解鎖方式 |
| **自動填入** | 在系統設定中將 Bitwarden 設為預設密碼自動填入服務 |
| **金庫逾時** | 設定閒置自動鎖定時間（建議 ≤ 15 分鐘） |

## CLI 工具

```bash
# 指定自建伺服器地址
bw config server https://vault.example.com

# 登入
bw login your_email@example.com
```

---

| 上一步 | |
|--------|-|
| [安全注意事項](precautions.md) | [返回 README](../README.md) |
