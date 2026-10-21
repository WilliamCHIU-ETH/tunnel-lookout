# 入隧前 · Tunnel Lookout 技術筆記

入隧前是一個幫用路人在進雪山隧道前，比較內外車道速度的路況 Web 服務。這個 repo 是它的技術筆記，說明為什麼做、怎麼運作、我在其中做了什麼。原始碼放在私人 repo，這裡只有文章和架構圖。

線上服務：<https://snow-tunnel-sentinel-edge.chiu-dev.workers.dev/>（網址沿用改名前的名稱）

## English summary

Tunnel Lookout is a traffic web service for drivers about to enter the Hsuehshan Tunnel on Taiwan's National Freeway 5. Lane changes are not allowed inside the tunnel, so the only chance to choose a lane is before entering. The service compares the speeds of the inner and outer lanes for the correct direction and section, shows how old each number is, and says "cannot tell" when the data is missing or stale, instead of guessing.

It runs on a Taiwan VPS (web server, Fastify API, a single data collector, and SQLite), with Cloudflare in front as the HTTPS entry point. The code was written mostly with AI coding tools (Codex, Claude Code) under my direction. These notes cover only the parts I can explain myself. The source code is private.

## 目錄

| 章節 | 內容 | 狀態 |
|---|---|---|
| [01 為什麼做](01-why.md) | 要解決的問題，以及「資料不足就說無法判斷」的原則 | 定稿 |
| [02 為什麼是台灣 VPS](02-why-taiwan-vps.md) | 從 Cloudflare 搬到台灣主機的判斷與代價 | 定稿 |
| 03 一個請求怎麼走 | 從瀏覽器到資料庫，經過哪些網路分層 | 撰寫中 |
| 04 活著不等於能用 | 健康檢查與「可用」檢查為什麼要分開 | 撰寫中 |
| 05 服務怎麼被管理 | systemd、log、監控與備份 | 撰寫中 |
| 06 一次部署被擋下的排查 | 部署保護機制誤擋的經過 | 撰寫中 |
| [07 分工與限制](07-ownership-and-limits.md) | 哪些是我做的、哪些是 AI 做的，以及還沒做到的事 | 定稿 |

03 到 06 章會在我實際動手驗證後才寫，只寫我能解釋的內容。

## 這份筆記怎麼寫

- **AI 協作**：程式碼、部署設定與大部分專案文件，是用 AI 開發工具在我的指示下完成的。第 07 章列出具體分工。
- **證據**：文中的系統行為以私人 repo 的文件與設定為準，並註明查核日期。沒有實際驗證過的事會明說。
- **不公開的內容**：主機位址、服務商帳號、SSH 設定、金鑰與 token、雲端儲存名稱、部署設定中的檔案指紋。

## 資料來源

| 資料 | 提供者 | 授權 |
|---|---|---|
| [VD 靜態資訊（v2.0）](https://data.gov.tw/dataset/37661)、[VD 動態資訊（v2.0）](https://data.gov.tw/dataset/37662)、[CCTV 靜態資訊（v2.0）](https://data.gov.tw/dataset/37665) | 交通部高速公路局 | [政府資料開放授權條款－第1版](https://data.gov.tw/license) |
| 國道五號排隊與事件資訊 | 交通部高速公路局 [1968 網站](https://1968.freeway.gov.tw/) | 讀取網站頁面公開使用的資料，不是開放資料集，沒有另外標示授權條款 |
| 天氣 | [Open-Meteo](https://open-meteo.com/) | [CC BY 4.0](https://open-meteo.com/en/licence)；Weather data by Open-Meteo.com |

授權條款於 2026-10-21 查核：三個政府資料集的頁面標示「政府資料開放授權條款-第1版」，Open-Meteo 授權頁標示 CC BY 4.0。

查核日期：2026-10-21，依私人 repo dev 分支 `a30d531`。

## 授權

本 repo 的文章與圖表以 [CC BY 4.0](LICENSE) 釋出，轉載或改寫請標示出處。上方列出的資料來源各自依其授權條款使用，不在此授權範圍內。
