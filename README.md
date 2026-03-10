# Bitwarden 自建伺服器部署專案

透過 Docker Compose 搭配 [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) 部署自建密碼管理伺服器，實現**零開放 Port、免暴露 IP** 的安全架構。

本專案提供**三套部署方案**，預設推薦 [Vaultwarden](https://github.com/dani-garcia/vaultwarden)（社群版、全功能免費、資源佔用最低）。

## 部署架構

```mermaid
graph TD
    subgraph "公網 Internet"
        A["🖥️ 各平台 Bitwarden 客戶端<br/>(瀏覽器 / 桌面 / 行動裝置 / CLI)"]
        B(("☁️ Cloudflare Edge Network<br/>WAF · DDoS 防護 · SSL 終止"))
    end

    subgraph "NAS / 伺服器主機（Docker Network: bw_net）"
        C["🔒 cloudflared 容器<br/>(Tunnel Connector)"]
        D["🗄️ Bitwarden 容器<br/>(Vaultwarden / Lite / Standard)"]
        E[("💾 SQLite 資料庫<br/>bw-data/")]
    end

    A -- "HTTPS 請求" --> B
    B -- "加密隧道<br/>Cloudflare Tunnel" --> C
    C -- "HTTP 反向代理" --> D
    D -- "讀寫" --> E

    classDef cloudflare fill:#F38020,stroke:#333,stroke-width:2px,color:#fff
    classDef local fill:#175DDC,stroke:#333,stroke-width:2px,color:#fff
    classDef client fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff
    classDef db fill:#6C757D,stroke:#333,stroke-width:2px,color:#fff

    class B cloudflare
    class C,D local
    class A client
    class E db
```

## 零知識加密模型

Bitwarden 採用**零知識架構（Zero-Knowledge Architecture）**——所有加解密作業完全在客戶端完成，伺服器端永遠不會接觸明文密碼。即使伺服器遭入侵，攻擊者取得的僅為無法解密的密文。

```mermaid
graph TB
    subgraph "客戶端（完全在本地執行）"
        U["使用者輸入明文密碼"]
        KDF["金鑰衍生<br/>PBKDF2-SHA256 (600K 次迭代)<br/>或 Argon2id"]
        MK["Master Key"]
        SK["Symmetric Key<br/>(AES-256-CBC)"]
        ENC["🔐 加密<br/>明文 → 密文"]
        DEC["🔓 解密<br/>密文 → 明文"]
        HASH["雜湊處理<br/>Master Password Hash<br/>(僅用於認證)"]
    end

    subgraph "伺服器端（僅儲存密文）"
        API["API 接收層"]
        DB[("💾 資料庫<br/>• 加密後密碼庫<br/>• 加密後對稱金鑰<br/>• 雜湊過的認證資料<br/><br/>❌ 無明文密碼<br/>❌ 無 Master Key")]
    end

    U --> KDF --> MK --> SK
    SK --> ENC
    MK --> HASH
    ENC -- "傳送密文 (TLS)" --> API --> DB
    DB -- "回傳密文 (TLS)" --> DEC
    SK --> DEC
    HASH -- "傳送雜湊 (TLS)" --> API

    classDef client fill:#4CAF50,stroke:#333,stroke-width:1px,color:#fff
    classDef server fill:#175DDC,stroke:#333,stroke-width:1px,color:#fff

    class U,KDF,MK,SK,ENC,DEC,HASH client
    class API,DB server
```

> 由於伺服器端不涉及解密，無論選擇哪套方案，密碼庫的加密安全性**完全相同**。

## 三套部署方案

| | ⭐ 社群版 Vaultwarden（預設） | 官方 Lite 版 | 官方標準版 |
|--|---------------------------|-------------|-----------|
| 映像 | `vaultwarden/server` | `ghcr.io/bitwarden/lite` | `ghcr.io/bitwarden/self-host/*` |
| 容器數量 | 1 | 1 | 10（微服務架構） |
| 維護方 | 社群 (dani-garcia) | Bitwarden Inc. | Bitwarden Inc. |
| 最低記憶體 | ~150 MB | 200 MB | 4 GB |
| 資料庫 | SQLite / MySQL / PG | SQLite / MySQL / PG | SQL Server 2022 |
| 需要 Installation ID | ❌ | ✅ | ✅ |
| 免費功能範圍 | 全功能免費 | 部分付費 | 部分付費 |
| Tunnel 內部 Port | `bitwarden:80` | `bitwarden:8080` | `bitwarden-nginx:8080` |
| 所在目錄 | **根目錄 `/`** | `lite/` | `standard/` |

> 詳細的硬體適配分析請參閱 [方案評估文件](docs/evaluation.md)。

## 專案結構

```
bitwarden-server/
├── docker-compose.yml          # ⭐ Vaultwarden（社群版，預設推薦）+ Cloudflared
├── .env.template               # Vaultwarden 環境變數模板
├── .gitignore
├── bw-data/                    # 持久化資料目錄
├── lite/                       # 官方 Bitwarden Lite（單一容器輕量版）
│   ├── docker-compose.yml
│   ├── settings.env
│   ├── .env.template
│   └── .gitignore
├── standard/                   # 官方標準版（微服務架構，10 容器）
│   ├── docker-compose.yml
│   ├── settings.env
│   ├── .env.template
│   └── .gitignore
└── docs/                       # 建置文件
    ├── evaluation.md           # 方案評估與硬體分析
    ├── deployment.md           # ⭐ Vaultwarden 部署步驟（預設）
    ├── lite-deployment.md      # 官方 Lite 部署步驟
    ├── standard-deployment.md  # 官方標準版部署步驟
    ├── cloudflare-tunnel.md    # Cloudflare Tunnel 設定教學
    ├── precautions.md          # 安全注意事項與備份策略
    └── client-setup.md         # 客戶端設定指南
```

## 文檔導覽

### 共用步驟

| 順序 | 文件 | 說明 |
|:----:|------|------|
| 0 | **[方案評估](docs/evaluation.md)** | DS224+ 硬體適配分析、加密架構說明 |
| 1 | **[Cloudflare Tunnel 設定](docs/cloudflare-tunnel.md)** | 建立隧道並取得 Token |

### 依選擇的方案繼續

| 方案 | 文件 |
|------|------|
| ⭐ Vaultwarden（預設推薦） | **[部署步驟](docs/deployment.md)** |
| 官方 Lite 版 | **[部署步驟](docs/lite-deployment.md)** |
| 官方標準版（需 ≥ 4GB RAM） | **[部署步驟](docs/standard-deployment.md)** |

### 部署後共用步驟

| 順序 | 文件 | 說明 |
|:----:|------|------|
| 3 | **[安全注意事項](docs/precautions.md)** | 關閉公開註冊、Admin 面板、備份策略 |
| 4 | **[客戶端設定](docs/client-setup.md)** | 跨平台 Bitwarden 客戶端連線至自建伺服器 |
