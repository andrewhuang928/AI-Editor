# 最終目標藍圖

```text
                    ┌──────────────┐
                    │   RAW VIDEO  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │  Downloader  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │   WhisperX   │
                    │              │
                    │ Transcript   │
                    │ Word Timing  │
                    └──────┬───────┘
                           ↓
                ┌────────────────────┐
                │       CLAUDE       │
                │                    │
                │ 內容理解           │
                │ 找 Hook            │
                │ 找精華             │
                │ 找 Cut             │
                │ 找 Zoom            │
                │ 找 B-roll          │
                │ 字幕               │
                │ 音效               │
                └─────────┬──────────┘
                          ↓
                 ┌─────────────────┐
                 │  YOUR STYLE     │
                 │                 │
                 │ Golden Samples  │
                 │ Style Rules     │
                 └────────┬────────┘
                          ↓
                 Editing Plan
                   ┌──────┼──────┐
                   ↓      ↓      ↓
                 Short1 Short2 ... Short5
                   └──────┼──────┘
                          ↓
                 ┌─────────────────┐
                 │ DaVinci Resolve │
                 │                 │
                 │ Cut             │
                 │ Subtitle        │
                 │ Zoom            │
                 │ B-roll          │
                 │ SFX             │
                 │ Music           │
                 │ Color           │
                 │ Animation       │
                 └────────┬────────┘
                          ↓
                    Render 5 Shorts
                          ↓
                    👤 YOU REVIEW
                    ↙    ↓    ↘
                  ❌     🔄     ✅
                       Re-edit   ↓
                                 📤
                               Publish

```

---

# 短期目標：Week 1 - Week 2 (9/3 - 9/17)

輸入一支原始講故事影片（Story 類別），**自動產出一條可在 DaVinci Resolve 裡繼續編輯的時間軸**（包含剪好的片段、變速、字幕），而不是一支拍平的成品 MP4。

人工僅負責語意層面的判斷（故事起訖點、整段刪除的離題內容、需保留的停頓），其餘機械性工作（如轉錄、剪點、字幕斷句、時間軸建構）皆交由系統自動化處理。

## 整體架構 (Pipeline)

```text
原始素材 (.mov)
    │
    ├─(有講稿?)─ 有 → 講稿輔助模式 (更準)
    │            無 → 純 ASR 模式 (其他類別走這條)
    ▼
① 轉錄 (WhisperX + hotwords 專有名詞偏置 + opencc 繁體修正)
    ▼
② 三層字詞校正
    第 1 層：事前引導 (hotwords，機會性)
    第 2 層：對照表自動替換 (已知錯字)
    第 3 層：LLM 語意校對 (有把握才自動改，其餘標記待確認)
    ▼
③ 人工輸入 (manual_input.json)
    - boundary：故事起訖
    - content_deletions：整段刪除的離題內容
    - preserved_pauses：刻意保留的停頓 (目前的缺口)
    - editorial：hook / title / reason
    ▼
④ 剪點規劃 (步驟C：靜音偵測 + 內容刪除 + 保留停頓)
    ▼
⑤ 字幕斷句規劃 (步驟D：長度先驗 + 詞邊界 + 靜音 + 講稿標點)
    ▼
⑥ editing_plan.json (黏合層，schema 驗證)
    ▼
⑦ 產出 SRT + OTIO
    ▼
⑧ 匯入 DaVinci Resolve (OTIO 建時間軸 + 變速 + SRT 字幕)
    ▼
輸出：可編輯的 Resolve 時間軸

```

## 目前確定可行、已驗證的技術路徑

| 環節 | 採用方案與備註 |
| --- | --- |
| **轉錄** | **WhisperX + hotwords**<br>

<br>（非 `initial_prompt`，避免幻覺回吐） |
| **繁體修正** | **opencc s2tw**<br>

<br>（非 s2twp，避免改變用詞） |
| **Resolve 時間軸建構** | **OTIO + LinearTimeWarp**<br>

<br>（放棄 `create_timeline_from_clips`，因該方法有 1-frame 間隙缺陷且無法變速） |
| **字幕匯入** | **SRT, Insert Selected Subtitles to → Timeline Using Timecode**<br>

<br>（目前唯一無副作用的方式；其他如直接匯入 Timeline 或拖曳皆有缺陷） |
| **雙 fps 換算** | 素材 60fps / 時間軸 30fps / 1.4倍速 → 除數為 2.8<br>

<br>（已使用 golden sample 62 段進行驗證） |
| **免費版 Resolve 自動化** | **in-app bridge**<br>

<br>（結構性限制：每次重開 Resolve 都必須重新啟動 bridge） |

## 目前進度追蹤 (Progress Tracking)

### ✅ 已完成並驗證 (Done)

* [x] 完整技術棧建置（WhisperX、Resolve MCP bridge、OTIO 匯出/匯入）。
* [x] 三層字詞校正機制實作。
* [x] 剪點規劃（步驟 C）與字幕斷句規劃（步驟 D）。
* [x] `editing_plan.json` 產生器及 schema 驗證。
* [x] SRT 產出（零偏移，不需補償）。
* [x] OTIO builder 測試通過（62 段可一次匯入，0 間隙、變速精確）。
* [x] **ep10（買櫝還珠）第一次完整跑通**：從素材到 Resolve 時間軸全部自動產出。
* [x] 講稿輔助模式評估完成並上線：證明講稿標點對字幕斷句效果顯著（Lift 11.39x，F1 Score 從 0.398 提升到 0.610）。

### ⏳ 尚未開始 (To-Do)

* [ ] **擴展至其他影片類別**：目前僅驗證過 Story 類別（且依賴講稿），需測試非 Story 類別的純 ASR 模式。
* [ ] **長影片選片邏輯**：Story 類別已確認不需選片（退化為邊界修剪），但其他長影片類別仍需開發此功能。
* [ ] **語意層的自動化開發**：目前的內容級刪除、保留停頓等判斷仍完全仰賴人工填寫。
* [ ] **風格層實作**：包含 Zoom-in、圖卡、音效 (SFX)、配樂、調色、片尾 CTA 等功能（目前完全空白）。
* [ ] **算圖 (Render) 自動化**。

---
