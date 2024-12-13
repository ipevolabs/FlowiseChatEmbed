# VC FlowiseChatEmbed 開發與發版流程

## 使用 vscode devcontainer 開發

1. 開啟 vscode 後，進入 Open in Container
2. 開啟終端機後，請先執行 `yarn dev` 啟動開發環境
3. 配置 git core editor 為 nano，`git config --global core.editor "nano"`
4. 在此環境下，進行 git commit 等操作
5. 在 github UI 上開 PR , squash & merge to dev branch

## 在 Host 端打 tag

```bash
[16:21:20] matt@mattmbp-2 /Users/matt/sdc1/ipevo/aiserver/trunk/vcFlowiseChatEmbed
> git tag 2.0.8.1
[16:21:28] matt@mattmbp-2 /Users/matt/sdc1/ipevo/aiserver/trunk/vcFlowiseChatEmbed
> git tag -l
2.0.8.1
[16:21:46] matt@mattmbp-2 /Users/matt/sdc1/ipevo/aiserver/trunk/vcFlowiseChatEmbed
> git push origin --tags
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To mattleei:ipevolabs/FlowiseChatEmbed.git
 * [new tag]         2.0.8.1 -> 2.0.8.1
[16:21:55] matt@mattmbp-2 /Users/matt/sdc1/ipevo/aiserver/trunk/vcFlowiseChatEmbed
>
```

## embed chatbot 使用的 URL

```html
<!-- public/index.html -->
<script type="module">
  const response = await fetch('https://raw.githubusercontent.com/ipevolabs/FlowiseChatEmbed/refs/heads/vc-release/vcbot-release.json');
  const config = await response.json();

  const chatbotModule = await import(`${config.jsUrl}?v=${config.version}`);
  const Chatbot = chatbotModule.default;
  Chatbot.init({
    chatflowid: '91e9c803-5169-4db9-8207-3c0915d71c5f', // Add your Flowise chatflowid
    apiHost: 'https://your-flowise-instance.com', // Add your Flowise apiHost
  });
</script>
```

## 開發流程 (in dev container)

1. 從 `dev` 分支建立新的功能分支

   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b feature/your-feature-name
   ```

2. 在功能分支上進行開發

   - 遵循開發規範進行程式碼撰寫
   - 完成功能後進行本地測試

3. 提交程式碼變更

   ```bash
   git add .
   git commit -m "feat: #123 實作新功能

   * resolved #123
   * feat
     * 實作新功能 A
     * 實作新功能 B
   * tests
     * 新增功能測試案例"
   ```

4. 建立 Pull Request

   - 從功能分支建立 PR 到 `dev` 分支
   - 填寫 PR 說明，包含功能描述、測試項目等
   - 請求程式碼審查

5. 程式碼審查與合併
   - 待程式碼審查通過
   - 解決審查意見（如果有）
   - 合併到 `dev` 分支

## 發版流程

1. 確認 `dev` 分支狀態

   - 確保所有要發布的功能都已合併到 `dev` 分支
   - 確保所有測試都已通過

2. 建立新版本標籤

   ```bash
   git checkout dev
   git pull origin dev
   git tag -a v2.0.8.3 -m "版本 2.0.8.3"
   git push origin v2.0.8.3
   ```

3. 更新發布設定

   版本設定檔，在 `vc-release` 分支上的 `vcbot-release.json` 檔案

   ```bash
   # 切換到 vc-release 分支
   git checkout vc-release
   git pull origin vc-release

   # 更新 vcbot-release.json
   # 修改檔案內容如下：
   {
     "jsUrl": "https://cdn.jsdelivr.net/gh/ipevolabs/FlowiseChatEmbed@2.0.8.3/dist/web.js",
     "version": "202403141200"  # 使用當前時間，格式：YYYYMMDDHHmm
   }

   # 提交變更
   git add vcbot-release.json
   git commit -m "chore: 更新發佈版本號到 2.0.8.3"
   git push origin vc-release
   ```

4. 驗證發布
   - 確認 CDN 上可以存取新版本
   - 測試客戶端是否可以正確載入新版本
   - 確認新功能是否正常運作

## 注意事項

- 版本號格式：`v主版本.次版本.修補版本.增量號碼`
- 時間戳格式：`YYYYMMDDHHmm`（年月日時分）
- 每次發版時都要更新 `version` 欄位，以確保客戶端會載入最新版本
- 發版前必須在測試環境完整測試
- 如果發現問題需要回滾，可以修改 `vcbot-release.json` 指向舊版本

## 相關資源

- CDN 位置：`https://cdn.jsdelivr.net/gh/ipevolabs/FlowiseChatEmbed`
- 設定檔位置：`https://raw.githubusercontent.com/ipevolabs/FlowiseChatEmbed/refs/heads/vc-release/vcbot-release.json`

## 問題排解

如果遇到以下問題：

1. CDN 快取問題

   - 等待 CDN 快取更新（通常需要 5-10 分鐘）
   - 可以使用 purge 功能強制更新 CDN 快取

2. 版本載入問題
   - 檢查 vcbot-release.json 內容是否正確
   - 確認 tag 是否已正確推送到 GitHub
   - 確認 CDN URL 是否正確
