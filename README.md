# motion-video — 程式算繪影片的 Claude Code skill

讓 Claude 用程式做一整支影片：每一幀都由程式依時間算出來（HTML / SVG / canvas 以時間驅動），逐幀截圖後用 ffmpeg 輸出 MP4。適用的片型：

- 介紹型廣告 (intro ad)、抓眼球的 hype 廣告 (hype ad)
- 產品導覽片 (product tour)
- 資料驅動的 showreel (data-driven showreel)
- 旁白說明動畫 (narrated explainer)、個人 / 作品集 reel (profile reel)

它管的是**整支片層級**的決策：先確認受眾、基調 (tone)、底色 (ground colour) 與這支片要讓人記住的一件事，再選片型骨架、決定換段節奏（跟音樂小節或跟旁白句首）、轉場、運鏡、產品真實性 (product fidelity)、輸出與逐幀審片。單一 UI 元件的 easing / spring 不在範圍內。

## 內容

| 路徑 | 說明 |
|---|---|
| `motion-video/SKILL.md` | 入口：工作順序、可以下的判定、交付時附的驗收清單 |
| `motion-video/references/` | 各主題規則：整支片規則、片型骨架、節奏 / 轉場 / 運鏡、建置與輸出、證據等級與量測工具、元件 spring 參數、「問產品負責人、別猜」的 prompt 附加段 |
| `motion-video/scripts/` | 16 支量測工具 (instruments)：拍點、換段、畫面動態比例、運鏡、形狀接續、底色切換、spring 擬合、螢幕文字與產品比對、contact sheet 等 |
| `motion-video/requirements.txt` | Python 套件 |

每一條規則都標了**證據等級 (evidence tier)**：

- **author**：作者看片後的判斷（一個人的品味紀錄；你自己的裁定優先）
- **measured**：校準過的量測工具在來源影片上量到的數字
- **verified**：經過盲測 A/B 回合、兩組範圍不重疊
- **unvalidated**：在來源影片量過，但還沒用模仿測試驗證

文中的 `Cnn`（來源影片）、`Tnn`（技法卡）、`rNN`（模仿回合）是作者私人實驗室的紀錄編號，只作出處標籤；實驗室本身沒有附上，附上的是其中通用的量測工具。

## 安裝

1. 把 `motion-video/` 整個資料夾複製到 Claude Code 的 skills 目錄：
   - Windows：`%USERPROFILE%\.claude\skills\motion-video\`
   - macOS / Linux：`~/.claude/skills/motion-video/`

   完成後應該看得到 `.../skills/motion-video/SKILL.md`。
2. 安裝 Python 套件（Python 3.10 以上）：

```powershell
pip install -r "$env:USERPROFILE\.claude\skills\motion-video\requirements.txt"
```

```powershell
python -m playwright install chromium
```

3. 確認 `ffmpeg` 與 `ffprobe` 在 PATH 上（`ffmpeg -version` 印得出版本即可）。
4. （選用）旁白片的量測需要逐字稿：自己用任何語音辨識工具產生 `.srt` 後以 `--srt` 傳入，或把環境變數 `MV_ASR_EXE` 設成一個接受 `<video> --out <dir> --language zh --stem transcript` 的語音辨識指令。
5. 重開 Claude Code。

## 驗證安裝

每支量測工具（`make_beat_track.py` 除外）都有自我測試，用合成的正例與反例確認它會觸發、也不會誤觸：

```powershell
cd "$env:USERPROFILE\.claude\skills\motion-video"
```

```powershell
python -X utf8 scripts\density.py --selftest
```

最後一行印出 `SELFTEST PASS` 就代表套件與 ffmpeg 都裝好了；其他工具用同樣方式測。

觸發測試：

- **應該觸發**：「幫我做一支 30 秒的產品宣傳片，要酷炫、抓眼球，放 IG」
- **不應該觸發**：「這個按鈕的 hover 動畫 easing 要怎麼調」（單一元件動效，不是整支片）

## 使用提醒

- skill 一開始會先問：片型、放在哪裡播、看完要觀眾做什麼、受眾、要記住的一件事、能量（平靜 / 活潑 / hype）、素材與限制。「介紹片」或「廣告」這幾個字不足以決定片型，它會問，不會猜。
- 產品畫面要用真實截圖，畫面上的每個數字都要追得到產品文件或實際畫面；它會交一份 `build/claims.json` 對照。
- 數字只能證明資料路徑，不能證明好看：「好不好看、像不像參考片」一律由你親自看片決定。

## 授權

MIT，見 [LICENSE](LICENSE)。審片流程的部分做法參考了 [reelmimic](https://github.com/edenfunf/reelmimic)，文中以 `[reelmimic]` 標註。
