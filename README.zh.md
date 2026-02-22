[English](README.md) | [繁體中文](README.zh.md)

# blink-builder

使用免費 Apple 個人團隊簽署，建置並側載 Blink Shell（iOS SSH/Mosh 終端機）。

## 概述

此技能自動化 **Blink Shell**（GPL 授權 iOS 終端機應用程式）的建置與安裝流程，使用免費 Apple 開發者簽署將其安裝到 iPhone 上。非常適合需要在 iOS 上使用 SSH/Mosh 但不想購買 Apple Developer Program 會員資格的開發者。

- **分支 repo**：[github.com/JonesHong/Blink-Shell-GPL-Builder](https://github.com/JonesHong/Blink-Shell-GPL-Builder)
- **上游**：[github.com/blinksh/blink](https://github.com/blinksh/blink)（GPL v3）
- **測試版本**：v18.3.0

## 快速開始

### 前置需求
- macOS 已安裝 Xcode
- Xcode 已登入 Apple ID
- iPhone 透過 USB 或同一 Wi-Fi 網路配對

### 一鍵安裝
```bash
git clone https://github.com/JonesHong/Blink-Shell-GPL-Builder.git
cd Blink-Shell-GPL-Builder
./setup.sh
```

## 主要指令

| 任務 | 指令 |
|------|---------|
| 健康檢查 | `./check.sh` |
| 重新安裝（相同版本） | `./reinstall-blink.sh` |
| 完整重建 | `./setup.sh` |
| 靜默重建（自動重新簽署） | `./reinstall-blink-auto.sh` |

## 功能特色

- **7 天憑證**：免費 Apple 簽署有效期為 7 天，到期需重新簽署
- **自動重新簽署**：可選設置 launchd 每 5 天自動重新簽署
- **零成本**：使用免費 Apple 個人團隊（無需開發者計劃費用）
- **Patch 管理**：透過自動 patch 偵測處理版本升級

## 限制

- **7 天憑證有效期** — 無付費帳號則無法繞過
- **每個免費 Apple ID 最多側載 3 個應用程式**
- **需要區域網路** — Mac 與 iPhone 必須在同一 Wi-Fi
- **需要設備信任** — 安裝後必須在設備上核准開發者

## 授權

此技能按現狀提供。Blink Shell 本身為 GPL v3。分支專為免費側載工作流程維護。
