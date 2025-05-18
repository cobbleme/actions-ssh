# 🎵 零成本使用 GitHub Actions + Serv00 部署 JMusicBot（Discord 音樂機器人）

> 靈感來源：[Yeecord 教學](https://yeecord.com/blog/make-discord-music-bot-without-coding)  
> [Yeecord 遇到的困難?](https://yeecord.com/blog/thats-why-i-gave-up-on-music)

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
2. 使用 SSH 連線進入 VPS，進入放置 bot 的資料夾（例如 `dc_bot/`）：

```bash
cd dc_bot
```

3. 第一次啟動 JMusicBot：

```bash
java -jar JMusicBot-x.x.x.jar
```

4. 輸入：
   - Discord Bot 的 **Token**
   - 你的 Discord 使用者 ID（右鍵個人頭像 → 複製使用者 ID）

5. 取得機器人邀請連結 → 加入你的伺服器。

6. （可選）按 `Ctrl + C` 停止程式後修改 `config.txt`、`serversettings.json` 等設定檔。

---

## 🧱 讓 Bot 持續背景運作（避免 GitHub Actions 時限）

GitHub Actions 最多只能執行 6 小時。若你希望 JMusicBot 在 VPS 上長時間運作，請使用以下指令將其掛在背景執行：

```bash
nohup java -jar JMusicBot-x.x.x.jar > /dev/null 2>&1 &
```

- `nohup`：保持程式在終端中斷後仍繼續運作。
- `> /dev/null 2>&1`：忽略所有輸出。
- `&`：讓程式在背景執行。

---

### 👀 查看背景執行狀態

使用以下指令查看正在執行的 Java 程序（包含 JMusicBot）：

```bash
ps aux | grep java
```

會顯示包含 `JMusicBot-x.x.x.jar` 的程序行。

---

### 🛑 停止背景中的 Bot

1. 找到對應的程序 PID。
2. 使用 `kill` 指令關閉：

```bash
kill <PID>
```

3. 若無法正常關閉，也可以強制終止：

```bash
kill -9 <PID>
```

---

## 🔐 產生並設置 SSH 金鑰（用於 GitHub Actions 登入 VPS）

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
          ssh -o StrictHostKeyChecking=no ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} "cd dc_bot && nohup java -jar JMusicBot-0.4.3.8.jar > /dev/null 2>&1 &"
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