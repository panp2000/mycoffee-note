# MyCoffee Note Web版 MVP 改訂開発計画仕様書

## 1. 改訂計画概要

### 1.1 概要と目的

**改訂理由**: 初期の7週間開発計画を見直し、初心者開発者がより実現可能な形に再構成するため。
**主な変更点**: 開発期間を10週間に延長し、実装ステップを明確化。初心者向けのガイダンスを強化。
**計画対象**: MyCoffee Note Web版MVPの開発全行程

### 1.2 改訂MVPの核心価値

- **抽出方法別の基本パラメータ記録**: 主要な抽出方法（ドリップ、エスプレッソなど）の基本パラメータを記録
- **シンプルな評価システム**: まずは星評価と基本メモから始め、後に拡張可能な設計
- **直感的なレシピ閲覧**: 基本的な検索・フィルター機能を持つシンプルなUI
- **信頼性の高いデータ保存**: Supabaseを活用したクラウドベースの安全なデータ管理
- **段階的なUI進化**: 機能性を優先し、徐々に洗練されたUIへと進化

### 1.3 機能優先順位の再定義

#### 最優先機能（Must Have）:
1. 基本的なユーザー認証（Eメール/パスワードのみ）
2. レシピの基本情報記録（コーヒー名、日付、メモ）
3. 抽出方法選択と基本パラメータ記録（6種類の主要抽出方法）
4. シンプルな評価機能（星評価とメモ）
5. レシピ一覧表示と基本検索

#### 次点優先機能（Should Have）:
1. レシピ編集機能
2. 基本的なフィルター機能
3. お気に入り機能
4. レスポンシブデザイン（モバイルファースト）
5. 抽出器具基本情報の記録

#### 省略可能機能（Could Have）:
1. Googleログイン連携
2. 詳細風味評価
3. タグ機能
4. アナリティクス
5. ダークモード

#### 後回し機能（Won't Have - MVP後）:
1. コミュニティ・共有機能
2. 高度な分析ダッシュボード
3. オフライン機能
4. バージョニング機能
5. データエクスポート/インポート

## 2. 改訂開発スケジュール（10週間）

### スプリント1（第1-2週）: プロジェクト基盤構築
**目標**: 開発環境の整備とプロジェクト基盤の構築

**主要タスク**:
- **日1-3**: プロジェクト初期設定
  - Vite + React + TypeScriptプロジェクト作成
  - 基本的な依存関係の設定（React Router等）
  - ESLint/Prettier設定
  - プロジェクト構造定義

- **日4-6**: Supabaseセットアップ
  - プロジェクト作成
  - 基本認証設定（Eメール/パスワード）
  - 基本的なテーブル設計・作成

- **日7-10**: 基本UIとルーティング
  - 基本レイアウトコンポーネント作成
  - メインルート設定
  - 認証ページの基本構造
  - シンプルなナビゲーション実装

- **日11-14**: 初期デプロイとテスト
  - Vercelへの初期デプロイ
  - CIパイプライン基本設定
  - 基本的なテスト環境整備
  - デプロイプロセスのドキュメント化

**成果物**: 
- 機能する開発環境とプロジェクト構造
- 基本的なSupabase連携
- 初期デプロイされたアプリケーション（白紙状態でも可）
- 基本的なルーティング構造

### スプリント2（第3-4週）: 認証と基本データモデル
**目標**: ユーザー認証システムと基本データモデルの実装

**主要タスク**:
- **日1-3**: 認証システム実装
  - サインアップフォーム
  - ログインフォーム
  - パスワードリセット機能
  - 認証状態管理

- **日4-6**: ユーザープロフィール基本機能
  - プロフィール表示ページ
  - 基本情報編集機能
  - 認証状態に基づくルート保護

- **日7-10**: 基本データモデル実装
  - レシピモデル定義
  - 抽出パラメータモデル定義
  - RLSポリシー設定
  - データアクセス基本関数

- **日11-14**: データサービスレイヤー構築
  - CRUD操作の基本実装
  - エラーハンドリング
  - 基本的なデータロード状態管理
  - テスト・検証

**成果物**:
- 機能するサインアップ/ログインフロー
- 基本的なユーザープロフィール管理
- レシピ関連のデータモデル実装
- 基本的なCRUD操作

### スプリント3（第5-6週）: レシピ基本機能
**目標**: レシピの作成・閲覧・編集の基本機能実装

**主要タスク**:
- **日1-3**: レシピ一覧表示
  - レシピカードコンポーネント
  - レシピリスト表示
  - 基本的なソート機能
  - 空の状態表示

- **日4-7**: レシピ作成フォーム
  - 基本情報入力フォーム
  - フォームバリデーション
  - 画像アップロード（任意）
  - データ保存機能

- **日8-10**: レシピ詳細表示
  - 詳細ビューレイアウト
  - データ取得と表示
  - 基本的なエラー処理
  - ローディング状態

- **日11-14**: 編集・削除機能
  - 編集フォーム実装
  - 削除確認ダイアログ
  - 機能のテストと改善
  - UX最適化

**成果物**:
- レシピのCRUD機能一式
- 機能するレシピ管理ワークフロー
- 基本的なユーザーインターフェース

### スプリント4（第7-8週）: 抽出方法パラメータと評価
**目標**: 抽出方法別パラメータと基本評価システムの実装

**主要タスク**:
- **日1-4**: 抽出方法セレクター
  - 抽出方法選択コンポーネント
  - 方法別アイコン/表示
  - 選択による条件分岐
  - 基本UIの統合

- **日5-8**: 抽出方法別パラメータフォーム
  - 動的フォームコンポーネント
  - 6種類の抽出方法実装
  - パラメータバリデーション
  - データ保存連携

- **日9-11**: 基本評価システム
  - 星評価コンポーネント
  - 評価入力フォーム
  - 評価表示
  - データ連携

- **日12-14**: フィルター機能とお気に入り
  - 基本的なフィルターUI
  - お気に入りトグル
  - フィルター適用ロジック
  - インタラクション改善

**成果物**:
- 抽出方法別のパラメータ入力
- 機能する評価システム
- 基本的なフィルタリングとお気に入り機能

### スプリント5（第9-10週）: UI改善とデプロイ
**目標**: UIの洗練、バグ修正、本番環境へのデプロイ

**主要タスク**:
- **日1-4**: UI改善
  - shadcn/uiコンポーネント導入
  - デザインの一貫性改善
  - アクセシビリティ対応
  - レスポンシブ対応強化

- **日5-8**: バグ修正と最適化
  - 既知の問題の修正
  - パフォーマンス最適化
  - エラー処理の強化
  - ユーザーフィードバック対応

- **日9-11**: テストとドキュメント
  - 手動テスト実施
  - 基本的なテストスクリプト作成
  - ユーザードキュメント作成
  - 開発者ドキュメント作成

- **日12-14**: 最終デプロイ
  - 本番環境構成
  - 最終デプロイ
  - デプロイ後テスト
  - ローンチ準備

**成果物**:
- 改善されたUI/UX
- バグ修正済みのアプリケーション
- 本番環境にデプロイされたMVP
- 基本的なドキュメント

## 3. 技術スタック導入ガイド

### 3.1 段階的導入アプローチ

#### フェーズ1: 基本スタック（スプリント1）
- **フロントエンド基盤**: 
  - Vite + React + TypeScript
  - 基本的なCSS（CSS Modules推奨）
  - React Router v6

- **開発環境**:
  - VS Code + 推奨拡張機能
  - ESLint + Prettier
  - Git (GitHub)

- **デプロイ**:
  - Vercel（GitHub連携）

#### フェーズ2: データと認証（スプリント2）
- **バックエンド**:
  - Supabase認証
  - Supabase Database
  - Supabase Storage（必要に応じて）

- **データアクセス**:
  - Supabase JavaScript Client
  - 基本的なfetch/axiosラッパー

#### フェーズ3: 拡張ライブラリ（スプリント3-5）
- **状態管理**:
  - React Context API → Zustand（必要に応じて）

- **フォーム処理**:
  - React Hook Form
  - Zod（バリデーション）

- **UIライブラリ**:
  - shadcn/ui（段階的に導入）

- **拡張機能**:
  - TanStack Query（データフェッチ最適化）
  - date-fns（日付操作）

### 3.2 Supabaseセットアップ手順

1. **Supabaseプロジェクト作成**:
   - [Supabase](https://supabase.com)でアカウント作成
   - 新規プロジェクト作成（無料枠で開始）
   - リージョン選択（日本に近いもの推奨）

2. **認証設定**:
   - Email認証の有効化
   - パスワードリセット設定
   - リダイレクトURL設定（開発/本番）

3. **データベーステーブル作成**:
   ```sql
   -- Usersテーブル (Supabase Authと連携)
   CREATE TABLE users (
     id UUID REFERENCES auth.users NOT NULL PRIMARY KEY,
     email TEXT NOT NULL,
     display_name TEXT,
     created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
     updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
   );

   -- RLSポリシー設定
   ALTER TABLE users ENABLE ROW LEVEL SECURITY;
   CREATE POLICY "ユーザーは自分のプロフィールを参照可能" ON users
     FOR SELECT USING (auth.uid() = id);
   CREATE POLICY "ユーザーは自分のプロフィールを編集可能" ON users
     FOR UPDATE USING (auth.uid() = id);

   -- Recipesテーブル (シンプル化)
   CREATE TABLE recipes (
     id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
     user_id UUID REFERENCES users(id) NOT NULL,
     name TEXT NOT NULL,
     coffee_name TEXT NOT NULL,
     is_favorite BOOLEAN NOT NULL DEFAULT false,
     created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
     updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
     origin TEXT,
     roast_level TEXT,
     notes TEXT
   );

   -- RLSポリシー設定
   ALTER TABLE recipes ENABLE ROW LEVEL SECURITY;
   CREATE POLICY "ユーザーは自分のレシピを操作可能" ON recipes
     FOR ALL USING (auth.uid() = user_id);

   -- BrewingParamsテーブル (シンプル化)
   CREATE TABLE brewing_params (
     id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
     recipe_id UUID REFERENCES recipes(id) NOT NULL,
     brewing_method TEXT NOT NULL,
     coffee_amount NUMERIC NOT NULL,
     water_amount NUMERIC NOT NULL,
     brewing_time INTEGER NOT NULL,
     water_temperature INTEGER,
     grind_size TEXT,
     method_specific_params JSONB DEFAULT '{}'::jsonb,
     created_at TIMESTAMPTZ NOT NULL DEFAULT now()
   );

   -- RLSポリシー設定
   ALTER TABLE brewing_params ENABLE ROW LEVEL SECURITY;
   CREATE POLICY "ユーザーは自分のパラメータを操作可能" ON brewing_params
     FOR ALL USING (
       EXISTS (
         SELECT 1 FROM recipes 
         WHERE recipes.id = brewing_params.recipe_id 
         AND recipes.user_id = auth.uid()
       )
     );

   -- 基本評価テーブル (シンプル化)
   CREATE TABLE evaluations (
     id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
     recipe_id UUID REFERENCES recipes(id) NOT NULL,
     user_id UUID REFERENCES users(id) NOT NULL,
     rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
     notes TEXT,
     created_at TIMESTAMPTZ NOT NULL DEFAULT now()
   );

   -- RLSポリシー設定
   ALTER TABLE evaluations ENABLE ROW LEVEL SECURITY;
   CREATE POLICY "ユーザーは自分の評価を操作可能" ON evaluations
     FOR ALL USING (auth.uid() = user_id);
   ```

4. **ストレージバケット設定**:
   - 「images」バケットの作成
   - RLSポリシーの設定

5. **クライアント接続情報**:
   - API URLとAnon Keyを取得
   - 環境変数に設定（.env.localなど）

### 3.3 フロントエンド初期設定手順

1. **プロジェクト作成**:
   ```bash
   # Viteプロジェクト作成
   npm create vite@latest mycoffee-note -- --template react-ts
   cd mycoffee-note
   npm install
   ```

2. **基本ライブラリインストール**:
   ```bash
   # ルーティングとスタイリング
   npm install react-router-dom
   
   # Supabase
   npm install @supabase/supabase-js
   
   # ユーティリティ
   npm install date-fns
   ```

3. **フォルダ構造セットアップ**:
   ```
   src/
   ├── components/       # 再利用可能なUI
   ├── pages/            # ページコンポーネント
   ├── hooks/            # カスタムフック
   ├── lib/              # ユーティリティ
   │   └── supabase.ts   # Supabaseクライアント
   ├── types/            # 型定義
   ├── assets/           # 静的アセット
   └── App.tsx           # メインアプリ
   ```

4. **Supabaseクライアント設定**:
   ```typescript
   // src/lib/supabase.ts
   import { createClient } from '@supabase/supabase-js';

   const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
   const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

   export const supabase = createClient(supabaseUrl, supabaseAnonKey);
   ```

5. **環境変数設定**:
   ```
   # .env.local
   VITE_SUPABASE_URL=あなたのSupabase URL
   VITE_SUPABASE_ANON_KEY=あなたのSupabase Anon Key
   ```

6. **基本ルーティング設定**:
   ```typescript
   // src/App.tsx
   import { BrowserRouter, Routes, Route } from 'react-router-dom';
   import HomePage from './pages/HomePage';
   import LoginPage from './pages/LoginPage';
   import SignupPage from './pages/SignupPage';
   import RecipesPage from './pages/RecipesPage';
   import Layout from './components/Layout';

   function App() {
     return (
       <BrowserRouter>
         <Routes>
           <Route path="/" element={<Layout />}>
             <Route index element={<HomePage />} />
             <Route path="login" element={<LoginPage />} />
             <Route path="signup" element={<SignupPage />} />
             <Route path="recipes" element={<RecipesPage />} />
           </Route>
         </Routes>
       </BrowserRouter>
     );
   }

   export default App;
   ```

## 4. 初心者向け実装ガイドとベストプラクティス

### 4.1 コーディングアプローチ

- **小さく始める**: 一度に1つの機能を実装。各機能は小さな単位に分割
- **頻繁にテスト**: 小さな変更ごとにテストし、問題の分離を容易に
- **段階的に複雑化**: シンプルな実装から始め、動作確認後に機能を追加
- **早期・頻繁なコミット**: 機能単位で小さくコミット、メッセージは明確に
- **コンソールログ活用**: デバッグにはconsole.logを積極的に使用
- **コメント付加**: 複雑なロジックには簡潔な説明コメントを追加

### 4.2 初心者がよく陥る落とし穴と対策

| 落とし穴 | 症状 | 対策 |
|---------|------|------|
| 過度な抽象化 | 早期に過剰な汎用化を図り複雑化 | 最初は具体的に実装し、パターンが見えてから抽象化 |
| 巨大コンポーネント | 1つのファイルが肥大化し管理困難に | 責務ごとに分割、300行を超えたら分割を検討 |
| 非同期処理の複雑化 | Promise処理の入れ子や複雑なチェーン | async/awaitの活用、エラー処理の早期実装 |
| 状態管理の混乱 | 複数の状態管理方法の混在 | 一貫した方法に統一、まずはuseStateから始める |
| スタイル競合 | CSS優先度の問題やスタイル上書き | CSS Modulesの活用、コンポーネント単位のスコープ |
| 型定義の不足 | TypeScript型エラーや不明確な型 | 早期からきちんとした型定義、anyの回避 |
| 遅すぎるリファクタリング | 技術的負債の蓄積 | 「動くコード→クリーンなコード」の反復 |

### 4.3 UI実装段階的アプローチ

#### フェーズ1: 機能するUI (スプリント1-2)
- シンプルなHTML構造と基本的なCSS
- 機能動作の確認を優先
- 基本的なレスポンシブ対応（Flexboxの活用）

#### フェーズ2: コンポーネント化 (スプリント3)
- 共通UIパターンの抽出とコンポーネント化
- 基本的なスタイルガイドの策定
- 再利用可能なフォームコンポーネント

#### フェーズ3: UIライブラリ統合 (スプリント4)
- shadcn/uiの段階的導入
- 一貫したテーマの適用
- アクセシビリティの強化

#### フェーズ4: 視覚的洗練 (スプリント5)
- アニメーションと遷移効果
- 微調整とポリッシュ
- エラー状態やエッジケースのUI改善

### 4.4 データモデル進化アプローチ

#### 基本モデル (スプリント1-2)
- User: 認証情報と基本プロフィール
- Recipe: レシピの基本情報
- BrewingParams: 抽出方法と基本パラメータ

#### 拡張モデル (スプリント3-4)
- Evaluation: 基本的な評価（星評価とメモ）
- Equipment: 基本的な器具情報（任意）

#### 将来の拡張モデル (MVP後)
- タグシステム
- 詳細風味プロファイル
- バージョニング
- 共有機能

## 5. チーム開発のためのガイドライン

### 5.1 ブランチ戦略

**シンプルなGitフロー**:
- `main`: 常にデプロイ可能な状態を維持
- `develop`: 開発の統合ブランチ
- 機能ブランチ: `feature/機能名`
- バグ修正: `fix/問題の簡潔な説明`

**コミットメッセージ規約**:
```
feat: レシピ一覧表示機能を追加
fix: 認証状態が保持されない問題を修正
refactor: レシピフォームコンポーネントをリファクタリング
docs: READMEを更新
```

### 5.2 コードレビュープロセス

- **セルフレビュー**: PRを出す前に自身でレビュー
- **ペアレビュー**: 可能であれば2人以上でレビュー
- **フォーカスポイント**: 
  - 機能性: 仕様通りに動作するか
  - 読みやすさ: コードは明確でわかりやすいか
  - エラー処理: 例外やエッジケースへの対応
  - パフォーマンス: 明らかな非効率はないか

### 5.3 ドキュメント管理

- **コードコメント**: 「なぜ」そうしたかの説明を重視
- **READMEの維持**: セットアップ手順と基本的な使い方を常に最新に
- **変更履歴**: 主要な変更点を記録
- **アーキテクチャ概要**: フォルダ構造と主要コンポーネントの説明

## 6. 推奨リソースとテンプレート

### 6.1 学習リソース

**初心者向けチュートリアル**:
- [React公式チュートリアル](https://reactjs.org/tutorial/tutorial.html)
- [TypeScript for JavaScript Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [Supabase認証ガイド](https://supabase.com/docs/guides/auth)
- [React Router入門ガイド](https://reactrouter.com/docs/en/v6/getting-started/tutorial)

**中級者向けリソース**:
- [React Hook Form ドキュメント](https://react-hook-form.com/get-started)
- [Zustand入門](https://github.com/pmndrs/zustand)
- [TanStack Query 基礎](https://react-query.tanstack.com/overview)
- [shadcn/ui コンポーネント](https://ui.shadcn.com/)

### 6.2 開発ツールとテンプレート

**必須ツール**:
- VS Code + 拡張機能:
  - ESLint
  - Prettier
  - TypeScript Error Translator
  - ES7 React Snippets
  - Tailwind CSS IntelliSense (shadcn/ui使用時)

**推奨テンプレート**:
- [Vite + React + TS テンプレート](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts)
- [Supabase認証テンプレート](https://github.com/supabase/supabase/tree/master/examples/auth/react)
- [React Hook Form 基本テンプレート](https://github.com/react-hook-form/react-hook-form/tree/master/examples)

**デバッグ支援**:
- [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) (Zustand連携可能)
- [Supabase CLI](https://github.com/supabase/cli)

## 7. 結論

この改訂開発計画は、MyCoffee Note Web版MVPを初心者開発者でも実現可能な形で構築するためのロードマップです。10週間の段階的アプローチにより、技術的な挑戦と学習曲線を管理しつつ、価値あるプロダクトを構築できます。

開発の成功は、完璧なコードや全機能の実装ではなく、ユーザーに価値を届け、そのフィードバックから学びを得ることにあります。この計画に従いながらも、ユーザーニーズに応じて柔軟に調整していくことが重要です。

---

改訂日: 2025年5月13日
