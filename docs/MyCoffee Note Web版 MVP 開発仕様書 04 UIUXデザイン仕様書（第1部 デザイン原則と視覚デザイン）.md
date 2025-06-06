# MyCoffee Note MVP UI/UXデザイン仕様書 (第1部: デザイン原則と視覚デザイン)

## 1. デザイン原則

### 1.1 主要デザイン原則
1. **シンプルさと明瞭さ**: 不要な要素を排除し、主要機能を直感的に把握できるデザイン
2. **階層的情報設計**: 重要な情報を常に表示し、詳細は必要に応じて表示する階層的構造
3. **一貫性**: 色、フォント、アイコン、操作感の一貫性を保つ
4. **フィードバック**: ユーザーの操作に対する明確なフィードバックを提供
5. **アクセシビリティ**: 様々なユーザーが利用できるインクルーシブなデザイン

### 1.2 ユーザビリティ目標
- 初回使用時の学習曲線を最小化
- 繰り返し使用する機能へのアクセスを効率化
- 情報入力の手間を最小限に抑える
- 視認性と可読性の最適化

## 2. ビジュアルデザイン

### 2.1 カラーパレット

**プライマリカラー**
- メインカラー: `#6F4E37` (コーヒーブラウン)
- アクセントカラー: `#D4A76A` (ラテカラー)

**セカンダリカラー**
- 背景色: `#FFFAF0` (クリーム色)
- コンテンツ背景: `#FFFFFF` (白)
- ボーダー: `#E0D8C0` (ライトベージュ)

**テキストカラー**
- 主要テキスト: `#333333` (ダークグレー)
- 副次テキスト: `#666666` (ミディアムグレー)
- 淡色テキスト: `#999999` (ライトグレー)

**フィードバックカラー**
- 成功: `#4CAF50` (グリーン)
- 警告: `#FFC107` (イエロー)
- エラー: `#F44336` (レッド)
- 情報: `#2196F3` (ブルー)

### 2.2 タイポグラフィ

**フォントファミリー**
- 基本フォント: `Roboto` / `San Francisco` (システムフォント)
- 代替フォント: `Helvetica Neue`, `Arial`, sans-serif

**フォントサイズ**
- 見出し大 (H1): 24px
- 見出し中 (H2): 20px
- 見出し小 (H3): 18px
- 本文: 16px
- 補足テキスト: 14px
- 小テキスト: 12px

**フォントウェイト**
- 見出し: Semi-Bold (600)
- 本文: Regular (400)
- 強調: Medium (500)
- ボタン: Medium (500)

### 2.3 アイコンと画像

**アイコンスタイル**
- Material Designアイコンセットを採用
- 線の太さ: 2px
- 角の丸み: 2px
- サイズ: 24px × 24px (標準)

**画像スタイル**
- レシピサムネイル: 16:9比率
- アバター: 円形
- 装飾画像: 柔らかな影と角丸処理

### 2.4 スペーシングとレイアウト

**基本単位**
- ベースユニット: 8px
- インターフェース要素間の余白: 16px (2×ベースユニット)
- セクション間の余白: 24px (3×ベースユニット)

**レイアウトグリッド**
- 12カラムグリッドシステム
- ガター幅: 16px
- コンテナパディング: 16px

**タッチターゲット**
- 最小サイズ: 44px × 44px
- 推奨間隔: 8px以上