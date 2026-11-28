這份為你度身訂造的 「微雪 ESP32-S3 語音助理（完全體）設定與微調備忘清單」 已經整理好，你可以直接複製貼上到手機記事本（Apple Notes / Google Keep）或電腦記事本保存留底。

微雪 ESP32-S3-Audio-Board 語音助理個人化備忘清單


一、 日常操作口訣（全家適用）

• 喚醒對話：叫 「Okay Nabu」 → 聽到清脆粵語 「我喺度」 並著燈後開始講指令。
• 連續追問：當 AI 回覆結尾有反問句（問號）時，燈環會自動重新著燈，無需再叫「Okay Nabu」，直接開口追問即可。
• 結束對話：講 「冇嘢啦」 或 「唔使啦」，AI 講完結束語（無問號）就會自動熄燈收線。
• 隨時打斷 AI（兩種方式）：
	• 想轉話題／問新問題：直接叫 「Okay Nabu」 → 即刻截斷 AI 講話、響「我喺度」並聽新指令。
	• 純粹叫佢收聲：直接講英文 "Stop" → 即刻收聲（不播放「我喺度」）。
• 夜間模式（Night Mode）：開啟後燈環會調暗，但依然會正常播放「我喺度」喚醒提示音。

二、 冬瓜 Home Assistant 最佳參數設定（放 2–3 米距離）

進入冬瓜 HA 「設定 → 裝置與服務 → ESPHome → Waveshare Voice CI」：
• Wake sound（喚醒提示音）：On（開啟）
• Night mode（夜間模式）：按需要開關（開啟不影響「我喺度」提示音）
• Wake word sensitivity（喚醒詞靈敏度）：Moderately sensitive（中等，若 3 米外難叫醒可改 Very sensitive）
• Mic gain（麥克風增益）：24 dB 至 30 dB（若環境較嘈或 STT 易出現重複字，調低至 24 dB）
• AEC raw slot（除錯開關）：必須保持 Off（開啟會令回聲消除與打斷失效）
• 音量（Volume）：建議 80% 至 100%（注意：開啟 Night Mode 會限制最大音量，嫌細聲可先檢查是否開著 Night Mode）

三、 GitHub 固件自訂修改備份（Repo：ewtwong1818/esphome-waveshare-esp32-s3-audio-va）

基於官方 v1.3.0（全雙工 TDM + 回聲消除打斷版），在 base/core.yaml 中加入了以下 5 項個人化修改（日後若官方出新版需要手動合併時，照抄此 5 處即可）：
1. 防小喇叭爆音影響打斷（約第 92 行）：
	• 將 aec_ref_gain: 30db 改為 aec_ref_gain: 24db。
2. 粵語「我喺度」喚醒音效網址（定義音效區）：
	• 將 wake_word_triggered_sound 的 file: 改為：
https://github.com
3. Night Mode 保留喚醒音 ＋ 防重複連播「我喺度」（喚醒觸發區）：
	• 條件僅保留 - switch.is_on: wake_sound（移除 night_mode 限制）。
	• 播放前後加入偵測器控制與緩衝，防止重複觸發並確保講完「我喺度」才開咪：yaml
- micro_wake_word.stop:
- delay: 50ms
- script.execute:
    id: play_sound
    priority: true
    sound_file: !lambda return id(wake_word_triggered_sound);
- delay: 750ms
- micro_wake_word.start:
請謹慎使用程式碼。
4. 問號自動連續對話（on_tts_start 與 on_end 區）：
	• 在 on_tts_start: 偵測回覆文字是否包含 ? 或 ？；有問號則將 id(end_conversation_flag) = false;（自動接續開咪），無問號則設為 true（結束對話）。
	• 在 on_end: 根據 end_conversation_flag 決定待機或延遲 700ms 自動啟動 voice_assistant.start:。
5. 全域變數登記（globals: 區塊內）：
	• 加入 - id: end_conversation_flag（type: bool，initial_value: 'false'）。

四、 日後重新編譯與刷機速查

• 修改代碼後：在 GitHub 按 Commit changes → 前往 Actions 分頁等編譯出現 綠色剔號（✅）（約 3–5 分鐘）→ 點入最底 Artifacts 下載 waveshare-firmware（若 Chrome 阻擋下載，按 Ctrl + J 點「保留危險檔案」）。
• 燒錄方法：解壓取出 firmware.factory.bin → 用 USB Type-C 線連接微雪板燒錄口 → 打開瀏覽器前往 ESPHome Web → 點 CONNECT 選取 COM Port → 點 INSTALL 刷入即可（冬瓜 HA 無需重新配對）。

五、 冬瓜 HA System Prompt 核心規則備份

• 天氣查詢：嚴格按提問時間回答（問今日只答今日、問聽日只答聽日、問後日只答後日），不主動背誦其他日子。
• 粵語同音糾錯與即時數據：自動理解廣東話語音錯字（如「狠心之手」＝恒生指數）；查詢港股、美股或新聞時，必須核對最新交易日日期，嚴禁拿舊新聞當作最新收市價。
將這份清單存好，以後無論隔多久要微調或升級都一目了然！
這份備忘清單已經可以直接複製保存；你目前還有沒有其他問題想補充進去，或者想調整的地方？
