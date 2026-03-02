# 松尾研究所スライド デザインシステム リファレンス

python-pptxを使用して松尾研究所テンプレート準拠のプレゼンテーションスライドを日本語で作成するための設計仕様。
テンプレート: `references/松尾研究所テンプレ_v2_DLしてお使いください.pptx` を直接使用する。

---

## 目次

1. [テンプレートの使い方](#テンプレートの使い方)
2. [レイアウト選択](#レイアウト選択)
3. [カラートークン](#カラートークン)
4. [タイポグラフィシステム](#タイポグラフィシステム)
5. [レイアウトグリッド](#レイアウトグリッド)
6. [スライドテンプレート](#スライドテンプレート)
7. [コンポーネント](#コンポーネント)
8. [チャートスタイリング](#チャートスタイリング)
9. [テーブルスタイリング](#テーブルスタイリング)
10. [フレームワークコンポーネント](#フレームワークコンポーネント)
11. [再利用可能コードパターン](#再利用可能コードパターン)

---

## テンプレートの使い方

### 基本ワークフロー

```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
from pptx.oxml.ns import qn

# テンプレートを開く
prs = Presentation('references/松尾研究所テンプレ_v2_DLしてお使いください.pptx')

# 既存スライド（サンプル9枚）を全削除
while len(prs.slides._sldIdLst) > 0:
    rId = prs.slides._sldIdLst[0].get(qn('r:id'))
    prs.part.drop_rel(rId)
    prs.slides._sldIdLst.remove(prs.slides._sldIdLst[0])

# レイアウトからスライドを追加
slide = prs.slides.add_slide(prs.slide_layouts[2])
slide.placeholders[0].text = "スライドタイトル"

# 保存
prs.save('output.pptx')
```

### テンプレートが自動提供するもの

スライドマスターにより以下が全スライドに自動適用される。**手動追加は不要**：
- ロゴ（右上）
- コピーライト（下部中央）「©︎MATSUO INSTITUTE, INC.」
- セパレーターライン
- テーマカラー定義

---

## レイアウト選択

| Layout# | 名前 | 用途 | 主なプレースホルダー |
|---------|------|------|---------------------|
| 0 | Title Slide | 表紙 | idx=0:タイトル, idx=1:日付, idx=10:部署, idx=11:宛名, idx=12:氏名 |
| 1 | Title and Content | コンテンツ（本文PH付き） | idx=0:タイトル, idx=1:本文, idx=12:ページ番号 |
| 2 | 1_Title and Content | コンテンツ（自由配置） | idx=0:タイトル, idx=12:ページ番号 ※本文PHなし |
| 3 | Custom Layout | セクション区切り | idx=0:タイトル（中央配置） |
| 4 | 2_Custom Layout | ブランク＋ページ番号 | idx=11:ページ番号 |
| 5 | 1_Custom Layout | ブランク＋ロゴ | なし |

### レイアウト選択コード

```python
LAYOUT_TITLE = 0        # 表紙
LAYOUT_CONTENT = 1       # 本文PH付きコンテンツ
LAYOUT_FREE = 2          # 自由配置コンテンツ（テーブル・チャート等）
LAYOUT_SECTION = 3       # セクション区切り
LAYOUT_BLANK_PAGENUM = 4 # ブランク＋ページ番号
LAYOUT_BLANK_LOGO = 5    # ブランク＋ロゴ
```

### プレースホルダーへのアクセス

```python
slide = prs.slides.add_slide(prs.slide_layouts[LAYOUT_TITLE])

# プレースホルダーの一覧を確認
for ph in slide.placeholders:
    print(f"idx={ph.placeholder_format.idx}, name='{ph.name}', type={ph.placeholder_format.type}")

# プレースホルダーにテキストを設定
slide.placeholders[0].text = "プレゼンテーションタイトル"
slide.placeholders[1].text = "2026/03/02"
slide.placeholders[10].text = "松尾研究所 MLシステム開発部門"
slide.placeholders[11].text = "○○ 御中"
slide.placeholders[12].text = "作成者名"
```

### プレースホルダーのテキスト書式設定

```python
from pptx.util import Pt
from pptx.dml.color import RGBColor

ph = slide.placeholders[0]
ph.text = ""  # クリア
tf = ph.text_frame
p = tf.paragraphs[0]
run = p.add_run()
run.text = "スライドタイトル"
run.font.name = "Hiragino Kaku Gothic Pro W6"
run.font.size = Pt(26)
run.font.bold = True
run.font.color.rgb = RGBColor(0x00, 0x00, 0x00)
```

---

## カラートークン

松尾研究所テンプレートのテーマカラーに準拠。全スクリプトの先頭で定数として定義する：

```python
from pptx.dml.color import RGBColor

# === 松尾研究所 カラーパレット ===
class C:
    # 背景
    bgWhite     = RGBColor(0xFF, 0xFF, 0xFF)
    bgLight     = RGBColor(0xF5, 0xF7, 0xFA)   # 交互行 / パネル背景
    bgNavy      = RGBColor(0x00, 0x20, 0x60)    # ヘッダーバー、強調背景

    # テキスト
    textBlack   = RGBColor(0x00, 0x00, 0x00)    # プライマリテキスト
    textDark    = RGBColor(0x33, 0x33, 0x33)    # 本文
    textMuted   = RGBColor(0x88, 0x88, 0x88)    # ソースライン、キャプション
    textWhite   = RGBColor(0xFF, 0xFF, 0xFF)

    # テーマカラー（松尾研テンプレート準拠）
    tx2         = RGBColor(0x44, 0x54, 0x6A)    # dk2 — ナンバーバッジ、セカンダリテキスト
    accent1     = RGBColor(0x5B, 0x9B, 0xD5)    # ライトブルー
    accent2     = RGBColor(0xED, 0x7D, 0x31)    # オレンジ
    accent3     = RGBColor(0xA5, 0xA5, 0xA5)    # グレー
    accent4     = RGBColor(0xFF, 0xC0, 0x00)    # イエロー
    accent5     = RGBColor(0x44, 0x72, 0xC4)    # ブルー（チャートプライマリ）
    accent6     = RGBColor(0x70, 0xAD, 0x47)    # グリーン

    # 派生カラー
    accent4Dark  = RGBColor(0xBF, 0x90, 0x00)   # ゴールドヘッダーバー
    accent4Light = RGBColor(0xFF, 0xD9, 0x66)   # ライトゴールドバー
    accent5Dark  = RGBColor(0x33, 0x4F, 0x93)   # テーブルヘッダー
    accent2Dark  = RGBColor(0xB2, 0x5C, 0x25)   # オレンジバー暗め
    tx2Dark      = RGBColor(0x33, 0x3D, 0x4F)   # バッジボーダー

    # アクセント
    accentRed   = RGBColor(0xC8, 0x10, 0x2E)    # アラート、ネガティブデルタ
    accentGreen = RGBColor(0x70, 0xAD, 0x47)    # ポジティブデルタ

    # 構造
    gridLine    = RGBColor(0xD0, 0xD0, 0xD0)    # テーブルボーダー、フレームワークライン
    separator   = RGBColor(0x00, 0x00, 0x00)    # タイトル下のセパレーターライン（3pt黒）

# Hex文字列版（テーマカラー参照用）
C_HEX = {
    "bgWhite": "FFFFFF", "bgLight": "F5F7FA", "bgNavy": "002060",
    "textBlack": "000000", "textDark": "333333", "textMuted": "888888",
    "tx2": "44546A", "accent1": "5B9BD5", "accent2": "ED7D31",
    "accent3": "A5A5A5", "accent4": "FFC000", "accent5": "4472C4",
    "accent6": "70AD47", "accent5Dark": "334F93", "accentRed": "C8102E",
    "gridLine": "D0D0D0",
}
```

---

## タイポグラフィシステム

松尾研究所テンプレートはHiragino Kaku Gothic Proファミリーを統一使用する。

| 役割 | フォント | サイズ | ウェイト | カラー | 用途 |
|------|---------|--------|---------|--------|------|
| スライドタイトル | Hiragino Kaku Gothic Pro W6 | 26pt | Bold | `C.textBlack` | コンテンツスライドのタイトル |
| タイトルスライド見出し | Hiragino Kaku Gothic Pro W6 | 36pt | Bold | `C.textBlack` | 表紙のメインタイトル |
| タイトルスライド大見出し | Hiragino Kaku Gothic Pro W6 | 28pt | Bold | `C.textBlack` | 表紙のクライアント名等 |
| 本文レベル1 | Hiragino Kaku Gothic Pro W3 | 24pt | Regular | `C.textBlack` | メインコンテンツ |
| 本文レベル2 | Hiragino Kaku Gothic Pro W3 | 20pt | Regular | `C.textBlack` | サブコンテンツ |
| 本文レベル3 | Hiragino Kaku Gothic Pro W3 | 16pt | Regular | `C.textBlack` | 詳細テキスト |
| 本文レベル4 | Hiragino Kaku Gothic Pro W3 | 12pt | Regular | `C.textBlack` | 注釈テキスト |
| データ数値 | Calibri | 10-12pt | Bold | `C.textBlack` | テーブル/チャート内の値 |
| ソースライン | Hiragino Kaku Gothic Pro W3 | 10.5pt | Regular | `C.textDark` | 「出典）...」データスライド下部 |
| ページ番号 | Hiragino Kaku Gothic Pro W3 | 12pt | Regular | `C.textBlack` | 右下 |
| コピーライト | Hiragino Kaku Gothic Pro W3 | 8pt | Regular | `C.textBlack` | 下部中央（テンプレート自動） |

### フォント設定ヘルパー

```python
from pptx.util import Pt
from pptx.dml.color import RGBColor

def set_font(run, name, size, bold=False, color=None):
    """runオブジェクトにフォント設定を適用"""
    run.font.name = name
    run.font.size = Pt(size)
    run.font.bold = bold
    if color:
        run.font.color.rgb = color

def set_slide_title_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W6", 26, bold=True, color=C.textBlack)

def set_body1_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W3", 24, color=C.textBlack)

def set_body2_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W3", 20, color=C.textBlack)

def set_body3_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W3", 16, color=C.textBlack)

def set_body4_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W3", 12, color=C.textBlack)

def set_source_font(run):
    set_font(run, "Hiragino Kaku Gothic Pro W3", 10.5, color=C.textDark)

def set_data_font(run):
    set_font(run, "Calibri", 10, bold=True, color=C.textBlack)
```

### フォントフォールバック戦略

Mac環境でHiragino Kaku Gothic Proが利用できない場合のフォールバック：
- Hiragino Kaku Gothic Pro W6 → "Hiragino Sans W6" → "Yu Gothic Bold"
- Hiragino Kaku Gothic Pro W3 → "Hiragino Sans W3" → "Yu Gothic" → "Meiryo UI"

Windows環境では：
- Hiragino Kaku Gothic Pro W6 → "Meiryo UI Bold" → "Segoe UI Bold"
- Hiragino Kaku Gothic Pro W3 → "Meiryo UI" → "Segoe UI"

---

## レイアウトグリッド

スライドサイズ: 13.33" × 7.5"（テンプレート準拠）

```
┌──────────────────────────────────────────────────────────┐
│ [Client名]                                    [Logo] │  y≈0.02（テンプレート自動）
│ ┌──────────────────────────────────────────────────────┐│
│ │ スライドタイトル (W6 26pt bold)                     ││  y=0.17, h=0.50
│ │ ━━━━━━━━━━━━━━━━━━━━━━ (3pt黒線・テンプレート自動)  ││  line y=0.67
│ │                                                     ││
│ │  コンテンツエリア                                   ││
│ │  (y=0.82 to y=6.8)                                 ││
│ │                                                     ││
│ │  利用可能: 11.95" 幅 × 5.98" 高                    ││
│ │                                                     ││
│ ├──────────────────────────────────────────────────────┤│
│ │ 出典）...                        ページ N          ││  y≈7.28
│ │            ©︎MATSUO INSTITUTE, INC.（テンプレート自動）│  y≈7.34
│ └──────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

### 座標定数

```python
from pptx.util import Inches

class L:
    # スライドサイズ
    slideW = Inches(13.33)
    slideH = Inches(7.5)

    # マージン
    mx = Inches(0.68)              # 左マージン（テンプレート準拠）
    my = Inches(0.17)              # 上マージン

    # タイトル
    titleX = Inches(0.68)
    titleY = Inches(0.17)
    titleW = Inches(11.95)
    titleH = Inches(0.50)

    # コンテンツエリア
    contentX = Inches(0.68)
    contentY = Inches(0.82)         # セパレーター下
    contentW = Inches(11.95)
    contentH = Inches(5.98)

    # フッター
    sourceX = Inches(0.68)
    sourceY = Inches(7.0)
    pageNumX = Inches(12.71)       # 右下
    pageNumY = Inches(7.28)

    # カラムヘルパー（コンテンツ幅11.95"基準）
    col2W = Inches(5.78)           # 2カラムレイアウトの各カラム幅
    col2Gap = Inches(0.4)
    col2RightX = Inches(6.86)      # 0.68 + 5.78 + 0.4

    col3W = Inches(3.72)           # 3カラムレイアウトの各カラム幅
    col3Gap = Inches(0.4)
    col3MidX = Inches(4.80)
    col3RightX = Inches(8.92)
```

---

## スライドテンプレート

### テンプレート: タイトルスライド（表紙）

Layout 0 を使用。プレースホルダーにテキストを設定する。

```python
def add_title_slide(prs, title, date=None, department=None, client_name=None, author=None):
    slide = prs.slides.add_slide(prs.slide_layouts[0])  # Title Slide

    # タイトル (idx=0)
    slide.placeholders[0].text = title

    # 日付 (idx=1)
    if date:
        slide.placeholders[1].text = date

    # 部署 (idx=10)
    if department:
        slide.placeholders[10].text = department

    # 宛名 (idx=11)
    if client_name:
        slide.placeholders[11].text = client_name

    # 氏名 (idx=12)
    if author:
        slide.placeholders[12].text = author

    return slide
```

### テンプレート: セクション区切りスライド

Layout 3 を使用。章タイトルのみを表示。

```python
def add_section_slide(prs, section_title):
    slide = prs.slides.add_slide(prs.slide_layouts[3])  # Custom Layout
    slide.placeholders[0].text = section_title
    return slide
```

### テンプレート: コンテンツスライド（本文PH付き）

Layout 1 を使用。箇条書き中心のコンテンツに最適。

```python
def add_content_slide(prs, title, body_text=None, page_num=None):
    slide = prs.slides.add_slide(prs.slide_layouts[1])  # Title and Content

    # タイトル (idx=0)
    slide.placeholders[0].text = title

    # 本文 (idx=1)
    if body_text:
        tf = slide.placeholders[1].text_frame
        tf.clear()
        for i, line in enumerate(body_text):
            p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
            run = p.add_run()
            run.text = line
            set_font(run, "Hiragino Kaku Gothic Pro W3", 16, color=C.textBlack)

    # ページ番号 (idx=12)
    if page_num is not None:
        slide.placeholders[12].text = str(page_num)

    return slide
```

### テンプレート: 自由配置コンテンツスライド

Layout 2 を使用。テーブル・チャート・シェイプを自由に配置する。

```python
def add_free_content_slide(prs, title, source_text=None, page_num=None):
    slide = prs.slides.add_slide(prs.slide_layouts[2])  # 1_Title and Content

    # タイトル (idx=0)
    slide.placeholders[0].text = title

    # ページ番号 (idx=12)
    if page_num is not None:
        slide.placeholders[12].text = str(page_num)

    # ソースライン（手動追加）
    if source_text:
        txBox = slide.shapes.add_textbox(L.sourceX, L.sourceY, Inches(8), Inches(0.3))
        tf = txBox.text_frame
        p = tf.paragraphs[0]
        run = p.add_run()
        run.text = f"出典）{source_text}"
        set_source_font(run)

    return slide
```

---

## コンポーネント

### テキストボックスの追加

```python
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR

def add_text_box(slide, x, y, w, h, text, font_name="Hiragino Kaku Gothic Pro W3",
                 font_size=16, bold=False, color=None, align=PP_ALIGN.LEFT,
                 valign=MSO_ANCHOR.MIDDLE):
    txBox = slide.shapes.add_textbox(x, y, w, h)
    tf = txBox.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.alignment = align
    run = p.add_run()
    run.text = text
    run.font.name = font_name
    run.font.size = Pt(font_size)
    run.font.bold = bold
    if color:
        run.font.color.rgb = color
    # 垂直方向の配置
    txBox.text_frame.paragraphs[0].space_before = Pt(0)
    txBox.text_frame.paragraphs[0].space_after = Pt(0)
    return txBox
```

### ヘッダーバー（矩形＋テキスト）

```python
from pptx.enum.shapes import MSO_SHAPE

def add_header_bar(slide, x, y, w, text, fill_color=None, text_color=None, h=None):
    bar_h = h or Inches(0.40)
    fill = fill_color or C.bgNavy
    txt_color = text_color or C.textWhite

    # 矩形
    shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, w, bar_h)
    shape.fill.solid()
    shape.fill.fore_color.rgb = fill
    shape.line.fill.background()  # 枠線なし

    # テキスト
    tf = shape.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.alignment = PP_ALIGN.LEFT
    run = p.add_run()
    run.text = text
    run.font.name = "Hiragino Kaku Gothic Pro W3"
    run.font.size = Pt(15)
    run.font.bold = True
    run.font.color.rgb = txt_color

    return shape
```

### ナンバーバッジ

四角形のナンバーバッジ（テンプレートではtx2カラー）。

```python
def add_number_badge(slide, x, y, num, size=None):
    s = size or Inches(0.50)

    # 四角バッジ
    shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, s, s)
    shape.fill.solid()
    shape.fill.fore_color.rgb = C.tx2
    shape.line.color.rgb = C.tx2Dark
    shape.line.width = Pt(1)

    # 番号テキスト
    tf = shape.text_frame
    p = tf.paragraphs[0]
    p.alignment = PP_ALIGN.CENTER
    run = p.add_run()
    run.text = str(num)
    run.font.name = "Hiragino Kaku Gothic Pro W3"
    run.font.size = Pt(16)
    run.font.bold = True
    run.font.color.rgb = C.textWhite

    return shape
```

### コールアウトボックス

矩形のコールアウト（テンプレートではoval不使用、矩形推奨）。

```python
def add_callout_box(slide, x, y, w, h, text, fill_color=None):
    fill = fill_color or C.accent4Light

    shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, w, h)
    shape.fill.solid()
    shape.fill.fore_color.rgb = fill
    shape.line.color.rgb = C.gridLine
    shape.line.width = Pt(1)

    tf = shape.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.alignment = PP_ALIGN.LEFT
    run = p.add_run()
    run.text = text
    run.font.name = "Hiragino Kaku Gothic Pro W3"
    run.font.size = Pt(14)
    run.font.color.rgb = C.textBlack

    return shape
```

---

## チャートスタイリング

python-pptxでチャートを追加する場合、`slide.shapes.add_chart()`を使用する。

### 棒グラフ（カラムチャート）

```python
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

def add_bar_chart(slide, categories, series_data, x=None, y=None, w=None, h=None):
    chart_x = x or L.contentX
    chart_y = y or L.contentY
    chart_w = w or L.contentW
    chart_h = h or Inches(4.5)

    chart_data = CategoryChartData()
    chart_data.categories = categories
    for name, values in series_data:
        chart_data.add_series(name, values)

    chart_frame = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED,
        chart_x, chart_y, chart_w, chart_h,
        chart_data
    )
    chart = chart_frame.chart

    # カラー設定
    colors = [C.accent5, C.accent1, C.accent4, C.accent2, C.accent6]
    for i, series in enumerate(chart.series):
        series.format.fill.solid()
        series.format.fill.fore_color.rgb = colors[i % len(colors)]

    return chart
```

### テーマカラーによるチャートカラー

python-pptxではschemeClr（テーマカラー）を使ってテンプレートのテーマに合わせることも可能。ただし直接操作はXML操作が必要なため、上記のRGBColor指定が安全：

```python
# テーマカラーのRGB値一覧（参照用）
CHART_COLORS = [
    C.accent5,    # 4472C4 — プライマリ
    C.accent1,    # 5B9BD5 — セカンダリ
    C.accent4,    # FFC000 — ターシャリ
    C.accent2,    # ED7D31 — クォータナリ
    C.accent6,    # 70AD47 — クイナリ
    C.accent3,    # A5A5A5 — グレー
]
```

---

## テーブルスタイリング

### 松尾研テンプレート準拠テーブル

```python
from pptx.util import Inches, Pt, Emu

def add_styled_table(slide, headers, rows, x=None, y=None, w=None, col_widths=None,
                     header_font_size=12, body_font_size=10):
    tbl_x = x or L.contentX
    tbl_y = y or L.contentY
    tbl_w = w or L.contentW
    n_rows = len(rows) + 1  # ヘッダー含む
    n_cols = len(headers)

    table_shape = slide.shapes.add_table(n_rows, n_cols, tbl_x, tbl_y, tbl_w, Inches(0.1))
    table = table_shape.table

    # カラム幅の設定
    if col_widths:
        for i, cw in enumerate(col_widths):
            table.columns[i].width = Inches(cw)

    # ヘッダー行
    for j, header in enumerate(headers):
        cell = table.cell(0, j)
        cell.text = header
        # ヘッダースタイル
        for paragraph in cell.text_frame.paragraphs:
            paragraph.alignment = PP_ALIGN.CENTER
            for run in paragraph.runs:
                run.font.name = "Hiragino Kaku Gothic Pro W3"
                run.font.size = Pt(header_font_size)
                run.font.bold = True
                run.font.color.rgb = C.textWhite
        # 背景色
        cell_fill = cell.fill
        cell_fill.solid()
        cell_fill.fore_color.rgb = C.accent5Dark

    # データ行
    for i, row in enumerate(rows):
        for j, cell_text in enumerate(row):
            cell = table.cell(i + 1, j)
            cell.text = str(cell_text)
            # スタイル
            for paragraph in cell.text_frame.paragraphs:
                paragraph.alignment = PP_ALIGN.LEFT if j == 0 else PP_ALIGN.LEFT
                for run in paragraph.runs:
                    run.font.name = "Hiragino Kaku Gothic Pro W3"
                    run.font.size = Pt(body_font_size)
                    run.font.color.rgb = C.textBlack
            # 交互背景色
            cell_fill = cell.fill
            cell_fill.solid()
            cell_fill.fore_color.rgb = C.bgWhite if i % 2 == 0 else C.bgLight

    return table
```

---

## フレームワークコンポーネント

### 2×2 マトリクス

```python
def add_2x2_matrix(slide, x, y, w, h, x_axis_label, y_axis_label, quadrants):
    """
    quadrants: [{"label": str, "description": str, "fill": RGBColor?}]
    順序: TL, TR, BL, BR
    """
    qw = (w - Inches(0.05)) / 2
    qh = (h - Inches(0.05)) / 2

    fills = [C.accent1, C.accent4Light, C.bgLight, C.accent5]
    positions = [
        (x, y),
        (x + qw + Inches(0.05), y),
        (x, y + qh + Inches(0.05)),
        (x + qw + Inches(0.05), y + qh + Inches(0.05)),
    ]

    for i, q in enumerate(quadrants):
        px, py = positions[i]
        fill_color = q.get("fill", fills[i])

        # 象限の矩形
        shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, px, py, qw, qh)
        shape.fill.solid()
        shape.fill.fore_color.rgb = fill_color
        shape.fill.fore_color.brightness = 0.5  # 50%透過相当
        shape.line.color.rgb = C.gridLine
        shape.line.width = Pt(0.5)

        # ラベル
        add_text_box(slide, px + Inches(0.15), py + Inches(0.12),
                     qw - Inches(0.3), Inches(0.35),
                     q["label"], "Hiragino Kaku Gothic Pro W6", 12,
                     bold=True, color=C.textBlack)

        # 説明
        if q.get("description"):
            add_text_box(slide, px + Inches(0.15), py + Inches(0.5),
                         qw - Inches(0.3), qh - Inches(0.65),
                         q["description"], "Hiragino Kaku Gothic Pro W3", 10,
                         color=C.textDark)

    # X軸ラベル
    add_text_box(slide, x, y + h + Inches(0.08), w, Inches(0.25),
                 x_axis_label, "Hiragino Kaku Gothic Pro W3", 10,
                 bold=True, color=C.textDark, align=PP_ALIGN.CENTER)

    # Y軸ラベル（回転はpython-pptxでは直接サポートされないため、テキストボックスを配置）
    add_text_box(slide, x - Inches(0.45), y, Inches(0.35), h,
                 y_axis_label, "Hiragino Kaku Gothic Pro W3", 10,
                 bold=True, color=C.textDark, align=PP_ALIGN.CENTER)
```

---

## 再利用可能コードパターン

### 完全なスライドスクリプト骨格

```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
from pptx.enum.shapes import MSO_SHAPE
from pptx.oxml.ns import qn

# === カラーパレット ===
class C:
    # ... 上記から貼り付け ...
    pass

# === レイアウト定数 ===
class L:
    # ... 上記から貼り付け ...
    pass

LAYOUT_TITLE = 0
LAYOUT_CONTENT = 1
LAYOUT_FREE = 2
LAYOUT_SECTION = 3

# === ヘルパー関数 ===
# ... set_font, add_text_box, add_header_bar, add_number_badge etc. ...

# === テンプレート関数 ===
# ... add_title_slide, add_section_slide, add_content_slide, add_free_content_slide ...

# === プレゼンテーション構築 ===
def build_deck():
    prs = Presentation('references/松尾研究所テンプレ_v2_DLしてお使いください.pptx')

    # 既存スライドを全削除
    while len(prs.slides._sldIdLst) > 0:
        rId = prs.slides._sldIdLst[0].get(qn('r:id'))
        prs.part.drop_rel(rId)
        prs.slides._sldIdLst.remove(prs.slides._sldIdLst[0])

    # スライド1: タイトル
    add_title_slide(prs,
        title="プロジェクト報告",
        client_name="○○ 御中",
        date="2026/03/02",
        department="松尾研究所 共同研究部門",
        author="担当者名"
    )

    # スライド2: セクション区切り
    add_section_slide(prs, "1. 分析結果")

    # スライド3: コンテンツ
    slide = add_free_content_slide(prs,
        title="デジタル売上は前年比34%増、主にエンタープライズ契約が牽引",
        source_text="社内データベース（2026年2月時点）",
        page_num=3
    )
    # テーブルやシェイプを追加
    # add_styled_table(slide, headers, rows)

    prs.save("output.pptx")
    print("プレゼンテーションを保存しました。")

if __name__ == "__main__":
    build_deck()
```

### 重要な注意事項

1. **テンプレートを必ず使用する** — `Presentation()` で空のプレゼンを作るのではなく、テンプレートファイルを開く。
2. **既存スライドを削除してから追加** — テンプレートにはサンプルスライドが含まれている。
3. **レイアウトを適切に選択** — 各スライドの用途に合ったレイアウトを使用する。
4. **ロゴ・コピーライト・セパレーターは手動追加しない** — スライドマスターが自動提供する。
5. **Hexカラーは `RGBColor` に変換** — `RGBColor(0x44, 0x72, 0xC4)` のように指定。
6. **スライドサイズはテンプレート準拠** — テンプレートが13.33" x 7.5"を定義済み。
7. **フォントはHiragino Kaku Gothic Pro** — W6（見出し）/ W3（本文）。Yu Mincho/Yu Gothicは不使用。
8. **ソース引用は「出典）」形式** — 10.5pt、スライド下部に配置。
9. **ナンバーバッジは四角形** — 円形ではなく、tx2カラー（44546A）の正方形。
10. **コールアウトは矩形推奨** — 楕円形ではなく矩形を使用。
11. **日本語テキストの幅に注意** — 日本語は英語より文字幅が大きいため、テキストボックスの幅に余裕を持たせる。
12. **実行方法** — `uvx --from python-pptx python generate-slides.py` で実行。
