# POI Showcase — 台北植物園

把台北植物園的旅遊資訊，用 `圖至SVG.py`（來源：`渲染邏輯/img2svg/圖至SVG.py`）的 **poi 模式**轉成
**一筆 POI 一個獨立 SVG**（不是一張合成地圖），共 **15 個 950×660 向量圖卡**。

原始內容：`台北植物園.md`（文字）與 `台北植物園.pdf`（排版版）。

## 目錄結構

```
poi-showcase/
├── data/pois.json          # 15 筆 POI 資料（分類、欄位、色票）＋總覽卡文案
├── svg/
│   ├── attractions/        # 旅遊景點 4
│   ├── transport/          # 交通 7
│   ├── restaurants/        # 素食餐廳 4
│   └── manifest.json       # 檔案清單 + bytes + viewBox
├── overview/
│   ├── overview.svg        # 單張總覽圖卡（1080×1440 向量）
│   ├── overview.png        # 同一張卡的點陣原圖（驗證用 ground truth）
│   ├── contact-sheet.png   # 15 張圖卡拼成的 5×3 總表（2092×1200）
│   └── manifest.json       # 總覽產出清單（尺寸、bytes）
├── 台北植物園.md
└── 台北植物園.pdf
```

## 產出清單（15 個 SVG，共 3.2 MB）

| 分類 | id | bytes |
|---|---|---|
| 旅遊景點 | `taipei-botanical-garden` | 270 KB |
| 旅遊景點 | `lotus-pond` | 174 KB |
| 旅遊景點 | `palm-avenue` | 169 KB |
| 旅遊景點 | `orchid-house` | 167 KB |
| 交通 | `mrt-xiaonanmen` | 218 KB |
| 交通 | `bus-botanical-garden` | 220 KB |
| 交通 | `bus-sanyuan-street` | 189 KB |
| 交通 | `bus-cksh` | 227 KB |
| 交通 | `to-liande` | 291 KB |
| 交通 | `to-hejia` | 216 KB |
| 交通 | `to-fanshuteng` | 237 KB |
| 素食餐廳 | `liande`（蓮德品素天地） | 238 KB |
| 素食餐廳 | `fanshuteng`（蕃薯藤自然食堂） | 219 KB |
| 素食餐廳 | `hejia`（禾甲蒸豆腐食堂） | 201 KB |
| 素食餐廳 | `three-idiots`（三個傻瓜印度蔬食，已停業） | 204 KB |

全部 viewBox 為 `0 0 950 660`，可直接當縮圖／卡片／社群圖使用。

## 重新產生

需要 PIL / numpy / scikit-learn 與 [Potrace](https://potrace.sourceforge.net/)：

```bash
python3 圖至SVG.py \
  -m poi \
  --data poi-showcase/data/pois.json \
  -o poi-showcase/svg \
  -c 12 -w 8
```

`-c` 為 K-Means 分色數（12 色），`-w` 為平行 worker 數。執行時間約 70 秒。

總覽卡（第二種模式，預設 1080×1440，可 `--width/--height` 覆寫）：

```bash
python3 圖至SVG.py \
  -m overview \
  --data poi-showcase/data/pois.json \
  -o poi-showcase/overview/overview.svg \
  --potrace-path /usr/sbin/potrace -c 16 -w 8
```

會同時輸出 `overview.png`（同尺寸 PIL 原卡），並印出每個區塊的 y 範圍與內容覆蓋率。

Contact sheet（把 `svg/` 下所有卡片拼成一張）：

```bash
python3 contact_sheet.py \
  poi-showcase/svg \
  poi-showcase/overview/contact-sheet.png \
  --cols 5 --cell-width 380
```

順序依 `svg/manifest.json`；沒有 manifest 時按檔名排序。`--gutter` / `--margin` / `--title` 可調。

## 三種圖怎麼選

- **要一眼掃完 15 張** → `overview/contact-sheet.png`（點陣，適合貼給人看）
- **要一張能縮放不失真的重點摘要** → `overview/overview.svg`（向量，1080×1440 直式）
- **要單一項目當縮圖／社群圖** → `svg/<分類>/<id>.svg`（向量，950×660 橫式）

## 流程

1. `data/pois.json` 每一筆 POI 先用 PIL 畫成 950×660 的卡片（標題、欄位、色塊）。
2. 卡片做 K-Means 分色，逐色層交給 Potrace 描成貝茲曲線。
3. 每層保留 Potrace 的 `<g transform="translate(0,H) scale(0.1,-0.1)">`，最後組成單一 SVG。

## 驗證

因為產出是「卡片 → 向量」的近似，用**逐像素比對**確認沒有走鐘：
把 PIL 原卡當 ground truth，用 chromium 截圖 SVG，比較 RGB 差異。

- 平均色差：**1.95 – 3.03 / 255**
- 差異面積：**1.8% – 3.1%**（集中在抗鋸齒邊緣）
- 渲染尺寸：15/15 精準 (950, 660)

總覽卡用同一套方法驗：

- 平均色差：**3.39 / 255**（門檻 12）
- 差異面積：**3.1%**
- 渲染尺寸：**(1080, 1440)** 精準
- 版面：內容覆蓋 62→1414px = **93.9% 畫布高度**，最長純空白橫帶 68px = 4.7%（門檻 15%）
- `overview.png` 與程式重建的原卡 **逐位元相同**（max diff 0）

Contact sheet 另檢查每一格非空白（縮圖後顏色數 > 20）：**15/15 有內容**。

## 已知注意事項

- Potrace 的 `<g transform=...>` **不能丟**。早期版本只抽 `<path>`，導致座標放大 10 倍且上下翻轉。
- 三個傻瓜印度蔬食已於 2026/11/30 停業，保留是為了比較表完整性。
- 資訊（營業時間、公車路線、價位）為撰寫當下查得，出行前請再確認。
