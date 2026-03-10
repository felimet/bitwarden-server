# 安全注意事項與備份策略

自建密碼管理伺服器涉及高度機敏資料的管理責任。本文件列出正式上線前必須完成的安全配置，以及持續營運中應遵循的維護準則。

---

## 1. 關閉公開註冊

預設情況下，任何知道伺服器網址的人都能自行註冊帳號。完成所有必要帳號的建立後，**務必立即關閉公開註冊功能**。

**操作方式：**

1. 編輯 `.env` 檔案：
   ```env
   SIGNUPS_ALLOWED=false
   ```
2. 重新啟動容器：
   ```bash
   docker compose down && docker compose up -d
   ```
3. 後續若需新增使用者，可透過 Admin 面板發送邀請信。

## 2. Admin 面板安全配置

Vaultwarden 提供 `/admin` 管理面板，用於管理使用者、組織與伺服器設定。

**啟用方式：**

1. 產生高強度 Token：
   ```bash
   openssl rand -base64 48
   ```
2. 將產生的字串填入 `.env`：
   ```env
   ADMIN_TOKEN=產生的隨機字串...
   ```
3. 重新啟動容器後，即可透過 `https://vault.example.com/admin` 存取管理面板。

> ⚠️ Admin Token 擁有伺服器的完全管理權限，請如同 Master Password 一般妥善保管。

## 3. 資料備份策略

> **⚠️ 若伺服器硬碟損壞且無備份，所有密碼資料將永久遺失。備份是自建伺服器最關鍵的責任。**

所有資料（SQLite 資料庫、附件、圖示快取等）均存放於 `bw-data/` 目錄中。

### 建議的備份方法

| 方法 | 頻率 | 說明 |
|------|------|------|
| **本地打包** | 每日 | `tar -czvf bitwarden-backup-$(date +%Y%m%d).tar.gz ./bw-data/` |
| **異地同步** | 每日 | 透過 Rclone / rsync 將備份檔自動傳輸至 NAS、AWS S3、Google Drive 等 |
| **客戶端匯出** | 每月 | 透過 Bitwarden Web Vault 匯出加密 JSON 檔案，離線保存 |

### 自動備份 Cron 範例

```bash
# 每日凌晨 3:00 自動備份 bw-data 並保留最近 30 天的備份
0 3 * * * tar -czvf /backup/bitwarden-$(date +\%Y\%m\%d).tar.gz /path/to/bw-data/ && find /backup/ -name "bitwarden-*.tar.gz" -mtime +30 -delete
```

## 4. SMTP 郵件通知

SMTP 非強制設定，但以下功能均依賴郵件服務：

| 功能 | 沒有 SMTP 的影響 |
|------|------------------|
| 新裝置登入通知 | ❌ 無法收到異常登入警告 |
| Admin 面板邀請使用者 | ❌ 無法發送邀請信 |
| 密碼重設 / 2FA 設定 | ❌ 部分流程無法完成 |
| 電子郵件驗證 | ❌ 無法驗證使用者信箱 |

常用的 SMTP 服務商：Gmail（應用程式密碼）、Mailgun、SendGrid、Amazon SES。

## 5. 定期更新容器映像檔

密碼管理器的漏洞修補至關重要，建議**至少每月更新一次**：

```bash
# 1. 先備份資料
tar -czvf bitwarden-backup-$(date +%Y%m%d).tar.gz ./bw-data/

# 2. 拉取最新映像檔並重新啟動
docker compose pull
docker compose down
docker compose up -d
```

## 6. 主機層級安全建議

| 項目 | 建議做法 |
|------|----------|
| 防火牆 | 僅開放 SSH (22)，無須開放 80/443（Cloudflare Tunnel 為 outbound 連線） |
| SSH 存取 | 停用密碼登入，改用 SSH Key 認證 |
| 自動更新 | 啟用主機作業系統的自動安全更新（`unattended-upgrades` 等） |
| Docker 權限 | 以非 root 使用者執行 Docker（加入 `docker` 群組即可） |

---

| 上一步 | 下一步 |
|--------|--------|
| [伺服器 Docker 部署](deployment.md) | [客戶端設定指南](client-setup.md) |
