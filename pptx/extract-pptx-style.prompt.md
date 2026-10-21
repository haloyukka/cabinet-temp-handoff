# プロンプト

````md
# 依頼：PowerPointのスタイル定義を抽出する

## あなたの役割
PowerPointテンプレートの構造とOOXMLに詳しいエンジニアとして作業してください。
目的は「このスタイル定義を見れば、同じ見た目のスライドを機械的に再現できる」状態にすることです。

## 入力ファイル
- テンプレート：【templates/xxx.potx】  ← 正（最優先）
- ガイドライン：【docs/xxx_guideline.pdf】
- 過去資料：【samples/*.pptx】  ← 実際の使われ方の参考

## 優先順位のルール
1. 内容が食い違う場合は「テンプレート > ガイドライン > 過去資料」の順で採用する。
2. 食い違いを見つけたら、採用しなかった側も含めて conflicts に記録する。
3. ファイルから確認できない値は推測で埋めず、`null` とし、unknowns に理由を書く。
4. テンプレートに含まれる要素は、使われていなさそうでもすべて定義する。

## 作業手順
目視ではなく、Pythonスクリプトで抽出してください（再実行できるようにするため）。

### ステップ1：棚卸し（ここで一度止まって報告）
- python-pptx と、pptxをzip展開したXML（theme*.xml、slideMaster*.xml、slideLayout*.xml）を読む。
- スライドサイズ、スライドマスター数、レイアウト一覧（名前・プレースホルダー数）を一覧で報告する。
- 抽出スクリプトは `scripts/extract_style.py` として保存する。

### ステップ2：抽出
次の項目を抽出する。
- スライドサイズ（EMUとcmの両方）
- テーマ：カラー（dk1, lt1, dk2, lt2, accent1〜6, hlink, folHlink をHEXで）、フォント（見出し・本文の英字／日本語）
- マスターのテキストスタイル：タイトル、本文レベル1〜5（フォント、サイズpt、太字、色、行間、段落前後の間隔、箇条書き記号、インデント）
- 各レイアウト：名前、プレースホルダーごとの種類・idx・位置とサイズ（EMU）・書式の上書き
- 固定要素：ロゴ、罫線、帯、フッター、ページ番号、日付、機密表示の位置と書式
- 表・グラフ・図形：過去資料で繰り返し使われている書式（色、線の太さ、塗り、フォントサイズ）
- ガイドラインに書かれたルール：余白、使用禁止事項、文字数の目安、表記ルールなど

色・フォントがテーマ参照（例：accent1）なのか直接指定（例：#1F4E79）なのかは区別して記録してください。

### ステップ3：出力
1. `style/style.yaml`：機械可読の正。各値に出典（source: template / guideline / sample:ファイル名）を付ける。
2. `style/STYLE_GUIDE.md`：人が読む説明書。style.yaml から生成した内容とし、独自の値を書き足さない。
3. `style/report.md`：conflicts（食い違い一覧）と unknowns（取得できなかった項目）。

## style.yaml の骨格
```yaml
meta:
  source_files: []
  extracted_at:
slide_size: { width_emu: , height_emu: , width_cm: , height_cm: }
theme:
  colors: { dk1: , lt1: , dk2: , lt2: , accent1: , ... }
  fonts:  { major_latin: , major_ea: , minor_latin: , minor_ea: }
text_styles:
  title: { font: , size_pt: , bold: , color: , align: , source: }
  body:
    level1: { font: , size_pt: , color: , bullet: , indent_emu: , line_spacing: , source: }
layouts:
  - name:
    placeholders:
      - { idx: , type: , x_emu: , y_emu: , w_emu: , h_emu: , overrides: {} }
fixed_elements: []
tables: {}
charts: {}
rules: []
conflicts: []
unknowns: []
```

## 守ってほしいこと
- 推測で値を補わない。わからない値は null と unknowns へ。
- 単位は必ず明記する（EMU / pt / cm）。
- 入力ファイルは変更しない。
- ステップ1の報告後、私の確認を待ってからステップ2に進む。
````
