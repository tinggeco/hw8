# Week 8 Assignment：ARIA v5.0 — 馬太鞍溪（三幕式）災害稽核系統

## 一、作業主題

本作業以 2025 年花蓮馬太鞍溪（Matai'an Creek）堰塞湖事件為案例，建立 ARIA v5.0（Automated Risk & Impact Audit）三幕式災害分析流程，利用 Sentinel-2 衛星影像進行災前（Pre-event）、事件中（Mid-event）與災後（Post-event）的完整稽核。

核心目標為：

1. 建立三時相（Pre → Mid → Post）事件證據鏈  
2. 偵測堰塞湖形成、崩塌源區與土石流堆積範圍  
3. 評估災害對光復鄉（Guangfu）與周邊設施之實際影響  
4. 建立可重現、可交付的災防分析工作流程

---

## 二、作業架構

### A. Three-Act STAC Scene Selection + TCI Quick-QA

使用 STAC API 搜尋 Sentinel-2 L2A 影像，依照事件發展選取：

- Pre-event：災前穩定森林狀態（無湖）
- Mid-event：堰塞湖形成階段（有湖）
- Post-event：潰決後土石流擴散階段

並以 TCI（True Color Image）進行快速人工 QA，確認場景可用性。

### B. Four Change Metrics（Three-Act aware）

建立多時相變化指標：

- NDWI（Normalized Difference Water Index）
- NBR（Normalized Burn Ratio）
- ΔNIR（近紅外變化）
- ΔSWIR（短波紅外變化）

作為後續災害分類與閾值判定基礎。

### C. Three Detection Masks with Tuned Thresholds

建立三種核心災害遮罩：

#### C1. Barrier Lake Mask（Pre → Mid）

偵測堰塞湖形成位置與面積。

#### C2. Landslide Source Scar（Pre → Post）

辨識上游崩塌源區。

#### C3. Debris Flow Footprint（Pre → Post）

辨識下游土石流堆積範圍。

### D. Multi-Layer Audit — Eyewitness Impact Table

將災害遮罩與資產圖層（shelters / nodes / facilities）疊合，建立：

## Impact Table（影響稽核表）

用於統計：

- 哪些設施被災害覆蓋
- Guangfu 實際受影響節點數量
- 高風險 shelter 分布情況

### E. AI Advisor Operational Brief（Bonus）

使用 Gemini 產生縣府災防指揮中心作戰簡報，模擬：

> 花蓮縣災害防救應變中心作業組組長  
> 向縣長提出 24 小時內決策建議

內容包含：

- 三幕式事件證據
- 潰決前預警窗口
- ARIA 覆蓋缺口
- 緊急調度命令
- 模型優化建議

---

## 三、環境設定（Infrastructure First）

### .env 環境變數

專案使用 `.env` 管理核心參數：

```env
STAC_ENDPOINT=
S2_COLLECTION=
MATAIAN_BBOX=
PRE_EVENT_START=
PRE_EVENT_END=
MID_EVENT_START=
MID_EVENT_END=
POST_EVENT_START=
POST_EVENT_END=
TARGET_EPSG=
AI_API_KEY=
```

此方式可避免將參數硬寫於 notebook 中，提高可重現性。

---

## 四、固定 Scene IDs（Reproducible Scene IDs）

為確保每次重跑結果一致，將三幕影像固定為：

```python
PRE_ITEM_ID
MID_ITEM_ID
POST_ITEM_ID
```

避免 STAC 每次回傳不同影像造成結果漂移。

---

## 五、AI Diagnostic Log（問題排查紀錄）

### 問題 1：Mid-event STAC 視窗只找到大量雲覆蓋影像

### 問題描述

在搜尋 Mid-event（堰塞湖形成期）影像時，原始時間窗回傳的 Sentinel-2 場景大多被厚雲覆蓋，無法辨識實際湖面位置，導致 barrier lake mask 無法穩定建立。

### 解決方式

我採取以下策略：

1. 將時間窗向前後各延伸數天
2. 使用 cloud cover 條件進一步篩選
3. 手動檢查 TCI（True Color Image）
4. 最終選定可清楚辨識湖面的場景作為 Mid-event scene

此方法可避免完全依賴自動雲量欄位，提升場景可靠性。

---

### 問題 2：Barrier lake mask 誤把河道陰影判定為湖泊

### 問題描述

初期使用 NDWI 閾值時，河道陰影與山谷陰影常被誤判為水體，造成大量 false positives。

### 解決方式

我使用以下方法進行修正：

1. 增加 NIR 條件限制
2. 排除狹長線型河道區域
3. 保留具封閉型湖面特徵的 polygon
4. 搭配人工視覺檢查確認結果

這使 barrier lake mask 更接近實際堰塞湖範圍，而非一般河道陰影。

---

### 問題 3：Landslide mask 在河床砂洲產生誤判

### 問題描述

Post-event 中部分裸露河床砂洲與崩塌裸露區光譜相似，造成 landslide mask false positives。

### 解決方式

使用：

```python
pre_NIR > 0.3
```

作為 vegetation gate。

只有災前具明顯植生覆蓋的區域，才允許被判定為 landslide scar。

此方法有效排除原本就裸露的河床區域，提高 landslide source 的可信度。

---

## 六、成果摘要

本次作業成功完成：

- 三幕式事件證據鏈建立
- 堰塞湖範圍辨識
- 崩塌源區與土石流區域辨識
- Guangfu 受災節點稽核
- AI 作戰簡報生成

使 ARIA v5.0 不僅能做「災害偵測」，更能做到：

## Operational Decision Support（作戰決策支援）

這也是本次作業最重要的核心價值。

---

