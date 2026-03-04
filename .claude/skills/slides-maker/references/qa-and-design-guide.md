# QA手順 & デザインガイド

スライド作成後の品質検査手順と、視覚的に優れたスライドを作るためのデザインアイデア集。

---

## 目次

1. [QA手順](#qa手順)
2. [画像変換](#画像変換)
3. [デザインアイデア](#デザインアイデア)

---

## QA手順

**前提: 問題があると想定して検査する。** 最初のレンダリングはほぼ間違いなく修正が必要。

### コンテンツQA

```bash
python -m markitdown output.pptx
```

- 内容の欠落、誤字、順序の誤りを確認
- テンプレートの残存プレースホルダーテキストを検出：

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

grepで結果が返ったら修正する。

### ビジュアルQA

**サブエージェントを使う** — コードを書いた本人は期待通りに見えてしまうため、フレッシュな目が必要。

スライドを画像に変換（[画像変換](#画像変換)参照）し、以下のプロンプトで検査：

```
Visually inspect these slides. Assume there are issues — find them.

Look for:
- Overlapping elements (text through shapes, lines through words, stacked elements)
- Text overflow or cut off at edges/box boundaries
- Decorative lines positioned for single-line text but title wrapped to two lines
- Source citations or footers colliding with content above
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
- Uneven gaps (large empty area in one place, cramped in another)
- Insufficient margin from slide edges (< 0.5")
- Columns or similar elements not aligned consistently
- Low-contrast text (e.g., light gray text on cream-colored background)
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
- Text boxes too narrow causing excessive wrapping
- Leftover placeholder content

For each slide, list issues or areas of concern, even if minor.

Read and analyze these images:
1. /path/to/slide-01.jpg (Expected: [brief description])
2. /path/to/slide-02.jpg (Expected: [brief description])

Report ALL issues found, including minor ones.
```

### 検証ループ

1. スライド生成 → 画像変換 → 検査
2. **発見した問題をリスト化**（問題ゼロなら、もっと厳しく見直す）
3. 問題を修正
4. **修正したスライドを再検証** — 1つの修正が別の問題を生むことがある
5. 全スライドで問題なしになるまで繰り返す

**最低1回の修正→再検証サイクルを完了するまで、完了と宣言しないこと。**

---

## 画像変換

スライドを個別画像に変換して目視検査する：

```bash
soffice --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

`slide-01.jpg`, `slide-02.jpg` 等が生成される。

修正後に特定スライドだけ再レンダリング：

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

### 依存ツール

- `pip install "markitdown[pptx]"` — テキスト抽出
- LibreOffice (`soffice`) — PDF変換
- Poppler (`pdftoppm`) — PDFから画像

---

## デザインアイデア

**退屈なスライドを作らない。** 白背景にプレーンな箇条書きでは印象に残らない。

### スライド設計の原則

- **コンテンツに合ったカラーパレットを選ぶ**: どのプレゼンに入れ替えても違和感がないパレットは、選択が甘い
- **色の優劣をつける**: 1色がドミナント（60-70%）、1-2色がサポート、1色がアクセント。均等配分にしない
- **明暗のコントラスト**: タイトル/結論スライドはダーク背景、コンテンツスライドはライト背景（サンドイッチ構造）
- **ビジュアルモチーフを統一**: 1つの特徴的な要素（角丸画像フレーム、色付き円内のアイコン、太い片側ボーダーなど）を選び、全スライドで繰り返す

### レイアウトパターン

**すべてのスライドにビジュアル要素が必要** — 画像、チャート、アイコン、シェイプのいずれか。テキストだけのスライドは記憶に残らない。

- 2カラム（左テキスト、右イラスト）
- アイコン＋テキスト行（色付き円内アイコン、太字ヘッダー、説明文）
- 2x2 / 2x3 グリッド
- ハーフブリード画像（左右いっぱい）＋コンテンツオーバーレイ
- 大きな数値コールアウト（60-72ptの数字＋小さいラベル）
- 比較カラム（Before/After、Pros/Cons）
- タイムライン / プロセスフロー（番号付きステップ、矢印）

### タイポグラフィ

**デフォルトのArialに頼らない。** ヘッダーに個性のあるフォント、ボディにクリーンなフォントをペアリング：

| ヘッダーフォント | ボディフォント |
|----------------|--------------|
| Georgia | Calibri |
| Arial Black | Arial |
| Calibri | Calibri Light |
| Cambria | Calibri |
| Trebuchet MS | Calibri |
| Impact | Arial |
| Palatino | Garamond |
| Consolas | Calibri |

| 要素 | サイズ |
|------|--------|
| スライドタイトル | 36-44pt bold |
| セクションヘッダー | 20-24pt bold |
| 本文 | 14-16pt |
| キャプション | 10-12pt muted |

**注意**: 松尾研究所テンプレート使用時は、`design-system.md` のタイポグラフィシステム（Hiragino Kaku Gothic Pro W6/W3）が優先される。上記は汎用ガイドとして参照。

### スペーシング

- 最小マージン 0.5"
- コンテンツブロック間 0.3-0.5"
- 余白を確保 — すべてのスペースを埋めない

### アンチパターン（避けるべきこと）

- **同じレイアウトの繰り返し** — カラム、カード、コールアウトをスライドごとに変える
- **本文のセンタリング** — 段落やリストは左揃え、タイトルのみセンター
- **サイズコントラスト不足** — タイトルは36pt以上で本文14-16ptとの差をつける
- **デフォルトの青** — トピックに合った色を選ぶ
- **不統一なスペーシング** — 0.3"か0.5"を選んで一貫させる
- **一部だけスタイル適用** — 全体を通してコミットする
- **テキストのみのスライド** — 画像、アイコン、チャート、ビジュアル要素を追加
- **テキストボックスのパディング忘れ** — シェイプと揃える際は`margin: 0`設定
- **低コントラスト要素** — アイコンもテキストも背景に対して十分なコントラストを確保
- **タイトル下のアクセントライン** — AI生成スライドの典型。ホワイトスペースか背景色で代用
