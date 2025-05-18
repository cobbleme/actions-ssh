# 🎵 零成本使用 GitHub Actions + Serv00 部署 JMusicBot（Discord 音樂機器人）

> 靈感來源：[Yeecord 教學](https://yeecord.com/blog/make-discord-music-bot-without-coding)

[Yeecord遇到的困難?](https://yeecord.com/blog/thats-why-i-gave-up-on-music)

這篇教學將帶你從 0 開始，利用免費的 GitHub Actions 搭配 Serv00 VPS，部署一個強大的 Discord 音樂機器人 —— [JMusicBot](https://github.com/jagrosh/MusicBot)。

---

## 🧰 準備事項

- 一個 GitHub 帳號
- 一台 VPS（推薦免費的 [Serv00](https://serv00.com)）
- 一個 Discord 帳號（用來建立機器人）

---

## 🛠️ Discord 機器人建立流程

1. 前往 [Discord Developer Portal](https://discord.com/developers/applications) 並登入。
2. 點選「**New Application**」，命名後建立。
3. 進入左側「**Bot**」頁面並點選「**Add Bot**」。
4. 建立完成後，**複製 Token**（非常重要！請妥善保存）。

---

## 🎶 下載 JMusicBot

1. 前往官方 JMusicBot 頁面下載 `.jar` 檔案：
   - 優先選用官方版本：[jagrosh/MusicBot](https://github.com/jagrosh/MusicBot/releases)
   - 若官方無法使用，也可以改用 Fork：[SeVile/MusicBot](https://github.com/SeVile/MusicBot/releases)

---

## ☁️ 上傳並執行 Bot 到 VPS（以 Serv00 為例）

1. 登入 Serv00 控制面板，使用檔案管理器將 `JMusicBot-x.x.x.jar` 上傳至 VPS。
2. 使用 SSH 連線進入 VPS，執行以下指令啟動 Bot：

```bash
java -jar JMusicBot-x.x.x.jar
```

3. 第一次啟動時會要求輸入：
   - Discord Bot 的 **Token**
   - 你的 Discord 使用者 ID（右鍵個人頭像 → 複製使用者 ID）
4. 之後會出現機器人邀請連結，點選並將其加入你的伺服器。
5. 最後按下 `Ctrl + C` 關閉程式。

---

## 🔐 產生並設置 SSH 金鑰

1. 在本地終端機執行以下指令產生一對 SSH 金鑰（無密碼）：

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions" -f gh-actions-key
```

2. 將公鑰加入 VPS 的 `~/.ssh/authorized_keys`：

```bash
cat gh-actions-key.pub >> ~/.ssh/authorized_keys
```

---

## 📦 GitHub 倉庫設置

1. 建立一個新的 GitHub 倉庫（建議設為私有）。
2. 進入「Settings → Secrets and variables → Actions」，新增以下三個 Secrets：

| 名稱            | 說明                             |
|-----------------|----------------------------------|
| `VPS_SSH_KEY`   | 貼上完整私鑰內容（含標頭與結尾） |
| `VPS_USER`      | VPS 使用者名稱（如 `root`）       |
| `VPS_HOST`      | VPS IP 或網域（如 `s4.serv00.com`）|

---

## ⚙️ 建立 GitHub Actions Workflow

1. 在 `.github/workflows/` 資料夾中新增一個 `deploy.yml` 檔案：

```yaml
name: One-Time VPS Command

on:
  workflow_dispatch:

jobs:
  ssh-run:
    runs-on: ubuntu-latest

    steps:
      - name: Set up SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.VPS_HOST }} >> ~/.ssh/known_hosts

      - name: Run command on VPS
        run: |
          ssh -o StrictHostKeyChecking=no ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} "echo '✅ VPS command succeeded!'; uptime"
```

---

## 🚀 執行你的 Workflow

1. 回到 GitHub 倉庫頁面，點選「Actions」。  
2. 點選左側的 `One-Time VPS Command`。
3. 右側按下「Run workflow」。  
4. 選擇分支（一般為 `main`），按下「Run workflow」即可觸發部署。

---

## ✅ 完成！

現在你已經成功用零成本架設了一個 Discord 音樂機器人！你可以自由修改啟動指令、部署腳本，甚至結合更多 GitHub Actions 自動化功能。

---

## 💬 歡迎交流與貢獻

如果你有更好的優化方式、遇到問題或想要一起改進這個專案，歡迎開 Issue 或發 PR！
