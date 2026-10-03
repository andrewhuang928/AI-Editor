# PROJECT_MAP.md — AI-Shorts-Editor 資料夾總覽

> 依 **2026-10-03 實際掃描磁碟**整理（不是照 `PROJECT_ARCHITECTURE.md` 的記載）。全專案約 **13.2 GB**，其中 **約 11.7 GB 是影片**（素材＋參考樣本），程式與設定加起來不到 1 MB。
> 剪好的 Resolve timeline **不在這個資料夾**（在 DaVinci Resolve 自己的資料庫），資料夾裡只有匯入用的 `outputs/*.otio`。**這個專案資料夾不是 git repo**（只有 `vendor/` 內有自己的 git），沒有版本控制。

## 1. 資料夾樹狀圖（兩層）

```
AI-Shorts-Editor/                        13.2G
├─ CLAUDE.md (30K)                       專案總規則，優先級最高
├─ PROJECT_ARCHITECTURE.md (688K)        實作細節＋版本紀錄；§0.0 是「接手起點」
├─ PROJECT_MAP.md                        本檔
├─ .mcp.json                             告訴 Claude 怎麼連 DaVinci Resolve（MCP 設定）
├─ scripts/ (27 支 .py, 556K)            引擎：轉錄、剪點、閘門、組計畫、建 OTIO（見 §2）
├─ templates/ (6.2G)                     「跨集共用」：風格規則、參考樣本、講稿庫
│  └─ Politician_Luo/
│     ├─ Story/ (146M)                   成語故事：風格規則、設定、講稿 story_p7-18.txt、1 組參考樣本
│     └─ History/ (6.1G)                 歷史故事：風格規則、設定、47 集講稿庫、3 組參考樣本
├─ projects/ (5.6G)                      「每一集」的工作區
│  └─ Politician_Luo/
│     ├─ Story/ (3.3G)                   成語故事 5 集：ep10、13、14、15、16
│     ├─ History/ (2.3G)                 歷史故事 2 集：ep21、ep22
│     ├─ 20260824_test/ (10M)            最早的測試專案（舊平面結構，未搬進 Story/）
│     └─ _resolve_tests/ (56K)           8/28–29 的 Resolve API 實驗與預測紀錄
├─ schemas/ (48K)                        editing_plan 的 JSON schema＋範例（validate_plan 用）
├─ config/ (4K)                          pinyin_confusions.json（同音字相似度設定）
├─ reference/ (12K)                      davinci-resolve-starter-prompt.md（最初的技術起始提示）
├─ vendor/ (88M)                         davinci-resolve-mcp：第三方 Resolve 控制程式（含自己的 git 與 venv）
├─ .venv/ (1.3G)                         專案的 Python 3.12 環境
└─ .claude/ (0B)                         空資料夾，**不確定**用途
```
**一集資料夾（`projects/…/<日期>_epN/`）裡面**：`raw/`（原始影片＋講稿 `script.txt`）、`transcript/`（逐字稿）、`outputs/`（`epN.otio`）、`manual_input.json`、`cuts_plan.json`、`captions_plan.json`、`editing_plan.json`、`media_probe.json`、`otio_seed.otio`、`vocab.json`、`_backup_*/`。

## 2. 分類

**A. 你會常常碰的**
| 檔案／資料夾 | 做什麼 |
|---|---|
| `projects/…/epN/raw/epN.MOV` | 你放進去的原始素材（**絕不覆蓋**） |
| `templates/Politician_Luo/History/transcript/` | 歷史故事講稿庫（EP016–EP066，47 個檔） |
| `templates/Politician_Luo/Story/story_p7-18.txt` | 成語故事講稿庫（只有第 7–18 集） |
| `projects/…/epN/manual_input.json` | **邊界、要刪的段落、標題**——我草擬、你確認（剪輯決策都在這） |
| `projects/…/epN/vocab.json` | 這一集的專有名詞（我草擬、你過目） |
| `EDITING_STYLE.md`（Story／History） | 風格規則，需要裁決風格時看 |
| Resolve 裡的 marker | 你聽完用 M 鍵標語助詞／口誤（資料在 Resolve，不在磁碟） |

**B. 系統自動產生、你不用管**
`transcript/*`（words.json 是正本，.srt/.tsv/.txt/.vtt 是副本）、`cuts_plan.json`、`captions_plan.json`、`editing_plan.json`、`media_probe.json`、`otio_seed.otio`、`outputs/*.otio`、`disfluency_candidates.json`、`_backup_*/`（每次改動前的備份）、`.DS_Store`、`scripts/__pycache__/`。

**C. 引擎和腳本（`scripts/`，27 支）**
| 流程步驟 | 腳本 |
|---|---|
| ① 轉錄 | `transcribe.py`（WhisperX → `words.json`） |
| ② 讀講稿 | `split_script.py`（Story 多集合一檔）、`read_episode_script.py`（History 單集檔） |
| ③ 剪點規劃 | `plan_cuts.py`（靜音＋你的刪除段 → `cuts_plan.json`） |
| ④ 匯入前閘門 | `quality_gate.py`（呼叫 `check_deletion_overreach.py`、`check_fragmentation.py`；有警示就擋） |
| ⑤ 組計畫＋驗證 | `plan_captions.py`（目前只為了 schema）、`build_editing_plan.py`、`validate_plan.py` |
| ⑥ 建 Resolve 匯入檔 | `build_otio.py` |
| 選用 | `list_disfluency_candidates.py`（口誤候選清單，只列不刪） |
| plan_captions 的小幫手 | `word_boundary.py`、`enclitic.py`、`script_punct.py`、`unit_start.py` |
| 舊字幕流程（ep13 起不用） | `build_srt.py`、`check_word_split.py` |
| 舊校對流程（已凍結） | `apply_corrections.py`、`check_corrections.py`、`layer3_correct.py`、`text_apply.py` |
| 風格逆向分析／開發期查證 | `audit_golden_sample.py`、`audit_removed_content.py`、`reconstruct_cuts_from_final.py`、`verify_bias.py` |
| 共用 | `pinyin_util.py`（同音字判定） |

**D. 設定和規則**
`CLAUDE.md`、`PROJECT_ARCHITECTURE.md`、兩份 `EDITING_STYLE.md`、兩份 `style_defaults.json`（變速、字幕時長、音訊、調色、渲染）、兩份 `cut_calibration.json`（剪點偏移補償）、兩份 style 層 `vocab.json`、`schemas/editing_plan.schema.json`、`config/pinyin_confusions.json`、`.mcp.json`、`templates/*/_templates/`（空白的 manual_input／vocab 範本）、`Story/corrections.json`＋`layer3_state.json`（舊校對設定，已凍結）。

## 3. 一集影片的資料流（🧑＝要你動手或確認，⚙️＝自動）

```
🧑 放 raw/epN.MOV（＋準備講稿）
 ⚙️ transcribe.py ───────────────► transcript/words.json
 ⚙️ split_script / read_episode_script ► raw/script.txt        （講稿→我挑專有名詞→🧑過目 vocab.json）
 🧑確認 ◄─ 我讀 words.json 草擬 ─► manual_input.json            （邊界／刪除段／標題）
 ⚙️ plan_cuts.py（words.json＋raw 音訊＋manual_input＋Story 或 History 的 style_defaults／cut_calibration）
                                  ─► cuts_plan.json
 ⚙️ quality_gate.py ─► 🧑 有警示就列給你確認，確認才放行
 ⚙️ plan_captions.py ─► captions_plan.json
 🧑 在 Resolve：開 project → 跑 resolve_bridge → 建 timeline → 拖入素材
 ⚙️ 我用 MCP 讀素材 fps ─► media_probe.json；建種子 ─► otio_seed.otio
 ⚙️ build_editing_plan.py ─► editing_plan.json ─► validate_plan.py（用 schemas/）
 ⚙️ build_otio.py ─► outputs/epN.otio ─► ⚙️ 匯入 Resolve＋四項驗證＋音軌逐段比對
 🧑 完整聽一遍，M 鍵成對標記語助詞／口誤 ─► ⚙️ 我讀回 marker ─► 🧑 確認範圍 ─► 更新 manual_input.json，從 plan_cuts 重跑
 （選用）⚙️ list_disfluency_candidates.py ─► disfluency_candidates.json ─► 🧑 聽候選
```

## 4. 清理建議（只列不動，請你決定）

| 項目 | 大小 | 判斷依據 | 建議 |
|---|---|---|---|
| `templates/…/History/golden_sample_01~03`（原片＋成品） | **6.1G** | 全專案最大；風格分析結果已存成 `reconstructed_cuts.json`＋逐字稿；但文件註明「原始素材保留」 | 最大的外接硬碟候選；移走前確認不再需要重跑分析 |
| `projects/…` 的 `raw/` 影片（ep10 1.5G、ep16 1.8G、ep21 0.85G、ep22 1.4G） | 5.5G | 是原片，CLAUDE.md 規定不得覆蓋；**Resolve 媒體連結指向這個路徑**，移走會離線（需 relink）。ep13／14／15 的 `raw/` 目前是空的（素材已不在原位） | 要移之前先想好 Resolve relink |
| 10 個 `_backup_*/`（Story ep14×2、History ep21×3、ep22×5） | 約 14M | 都是修改前的備份；EP21 的 `…_REVERTED` 是已放棄的嘗試；EP22 已定案 | 太小不急；EP21 的 REVERTED 最安全可刪 |
| 30 個 `.DS_Store`、`scripts/__pycache__/` | < 1M | macOS／Python 自動產生 | 可安全刪 |
| `_resolve_tests/srt_probe/`（3 個 SRT） | < 1K | 文件完全沒提到 | 可歸檔；`_resolve_tests/` 其餘文件有引用為證據，**留著** |
| `projects/…/20260824_test/` | 10M | 最早測試專案，**文件提到 14 次**（校準數字出處；Story `render.output_path` 還指向它） | 暫勿動 |
| `.claude/` | 0B | 空資料夾 | **不確定**用途 |
| 舊流程腳本與設定（校對、字幕、逆向分析） | 約 0.1M | 目前流程用不到，但文件仍引用 | 不佔空間，不建議動 |

## 5. 容易搞混的地方

| 名稱 | 差在哪 |
|---|---|
| `templates/` vs `projects/` | `templates/`＝**跨集共用的規則、參考樣本、講稿**；`projects/`＝**每一集自己的工作檔** |
| Story vs History 的 `style_defaults.json` | 變速都是 1.15、`source_form` 相同、校準數值相同（History 借用 Story）；差在字幕時長約束（Story 有、History 是 null）、音訊增益、調色、審核關卡、輸出路徑。**都不影響剪點**；Story 的 `render.output_path` 是舊測試路徑 |
| 兩份 `vocab.json` | `templates/…/vocab.json`＝**風格層**（跨集共用，很小）；`projects/…/epN/vocab.json`＝**這一集**；轉錄兩個都要帶 |
| 兩份 `EDITING_STYLE.md` | Story（1,752 行，只靠 1 支參考樣本）vs History（146 行，3 支，可信度各不同） |
| `manual_input.json` vs `_templates/manual_input.skeleton.json` | 前者是這一集真正的決策檔；後者是空白範本 |
| 三個 `*_plan.json` | `cuts_plan`＝剪點；`captions_plan`＝字幕斷句（目前只為通過 schema）；`editing_plan`＝把前兩者加風格組成的總計畫 |
| `words.json` vs `.srt/.tsv/.txt/.vtt` | 只有 `words.json` 被腳本讀；其餘是給人看的副本 |
| `transcript_raw` vs `transcript_final`（參考樣本內） | raw＝原始素材逐字稿；final＝成品影片逐字稿 |
| `CLAUDE.md` vs `.claude/`；兩個 `CLAUDE.md` | 根目錄的是本專案規則；`.claude/` 是空資料夾；`vendor/davinci-resolve-mcp/CLAUDE.md` 是第三方自己的 |
| 兩個 Python 環境 | `.venv/`（本專案腳本用）vs `vendor/davinci-resolve-mcp/venv/`（Resolve MCP 用，`.mcp.json` 指向它） |
| `CLAUDE.md` 的 §編號 vs `PROJECT_ARCHITECTURE.md` 的 §編號 | **兩套獨立編號**，不要互相對應 |
| 磁碟上的 `epN.otio` vs Resolve 裡的 `epN` timeline | 磁碟上的是匯入用檔案；可編輯的 timeline（含舊版與聽音 timeline）只存在 Resolve 資料庫 |
