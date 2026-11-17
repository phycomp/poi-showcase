# POI Showcase — 台北植物園

把台北植物園的旅遊資訊，用 `圖至SVG.py`（來源：`渲染邏輯/img2svg/圖至SVG.py`）的 **poi 模式**轉成
**一筆 POI 一個獨立 SVG**（不是一張合成地圖），共 **15 個 950×660 向量圖卡**。

原始內容：`台北植物園.md`（文字）與 `台北植物園.pdf`（排版版）。

## 目錄結構

```
poi-showcase/
├── data/pois.json          # 15 筆 POI 資料（分類、欄位、色票）
├── svg/
│   ├── attractions/        # 旅遊景點 4
│   ├── transport/          # 交通 7
│   ├── restaurants/        # 素食餐廳 4
│   └── manifest.json       # 檔案清單 + bytes + viewBox
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

## 已知注意事項

- Potrace 的 `<g transform=...>` **不能丟**。早期版本只抽 `<path>`，導致座標放大 10 倍且上下翻轉。
- 三個傻瓜印度蔬食已於 2026/11/30 停業，保留是為了比較表完整性。
- 資訊（營業時間、公車路線、價位）為撰寫當下查得，出行前請再確認。
