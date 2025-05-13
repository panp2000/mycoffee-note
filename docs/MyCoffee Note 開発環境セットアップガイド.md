# MyCoffee Note 開発環境セットアップガイド

このガイドでは、MyCoffee Note Web版MVPの開発環境を初めから設定する手順を説明します。初心者開発者向けに、各ステップを詳細に解説しています。

## 目次

1. [前提条件](#1-前提条件)
2. [開発ツールのインストール](#2-開発ツールのインストール)
3. [プロジェクトの作成](#3-プロジェクトの作成)
4. [依存関係のインストール](#4-依存関係のインストール)
5. [Supabaseのセットアップ](#5-supabaseのセットアップ)
6. [プロジェクト設定](#6-プロジェクト設定)
7. [Vercelへのデプロイ設定](#7-vercelへのデプロイ設定)
8. [トラブルシューティング](#8-トラブルシューティング)

## 1. 前提条件

始める前に、以下が必要です：

- 基本的なHTMLとCSSの知識
- JavaScriptの基本的な知識
- GitHubアカウント
- インターネット接続

特別な前提知識がなくても、手順に従うことで開発環境をセットアップできます。

## 2. 開発ツールのインストール

### 2.1 Node.jsのインストール

Node.jsは、JavaScriptをサーバーサイドで実行するための環境です。

1. [Node.js公式サイト](https://nodejs.org/)にアクセス
2. 推奨版（LTS）をダウンロード（16.x以上）
3. インストーラーを実行し、指示に従ってインストール

インストールを確認するには、ターミナル（コマンドプロンプト）を開いて以下のコマンドを実行します：

```bash
node -v
npm -v
```

バージョン番号が表示されれば、正常にインストールされています。

### 2.2 Visual Studio Codeのインストール

Visual Studio Code（VS Code）は、コードエディタです。

1. [VS Code公式サイト](https://code.visualstudio.com/)にアクセス
2. インストーラーをダウンロード
3. インストーラーを実行し、指示に従ってインストール

### 2.3 推奨VS Code拡張機能のインストール

VS Codeを起動し、左側のExtensionsタブ（または Ctrl+Shift+X）から以下の拡張機能をインストールします：

- ESLint
- Prettier - Code formatter
- ES7+ React/Redux/React-Native snippets
- TypeScript Error Translator
- GitLens

### 2.4 Gitのインストール

Gitはバージョン管理システムです。

1. [Git公式サイト](https://git-scm.com/downloads)にアクセス
2. お使いのOSに合わせてインストーラーをダウンロード
3. インストーラーを実行し、基本的なデフォルト設定でインストール

インストールを確認するには、ターミナルで以下のコマンドを実行します：

```bash
git --version
```

## 3. プロジェクトの作成

### 3.1 GitHubリポジトリの作成

1. [GitHub](https://github.com/)にログイン
2. 右上の「+」ボタン → 「New repository」をクリック
3. リポジトリ名に `mycoffee-note` を入力
4. 「Add a README file」にチェック（任意）
5. 「Create repository」をクリック

### 3.2 リポジトリのクローン

ターミナルを開き、作業したいディレクトリに移動して以下のコマンドを実行します：

```bash
# あなたのGitHubユーザー名に置き換えてください
git clone https://github.com/あなたのユーザー名/mycoffee-note.git
cd mycoffee-note
```

### 3.3 Vite + React + TypeScriptプロジェクトの作成

以下のコマンドでViteを使用してReact + TypeScriptプロジェクトを作成します：

```bash
npm create vite@latest . -- --template react-ts
```

`.`（ドット）を指定することで、現在のディレクトリに直接プロジェクトを作成します。途中で質問が表示された場合は、指示に従って回答してください。

## 4. 依存関係のインストール

### 4.1 基本依存関係のインストール

以下のコマンドで基本的な依存関係をインストールします：

```bash
# プロジェクトの依存関係をインストール
npm install

# ルーティングとSupabase
npm install react-router-dom @supabase/supabase-js

# 日付操作ライブラリ
npm install date-fns
```

### 4.2 開発用依存関係のインストール

```bash
# 開発ツール（ESLint, Prettier）
npm install -D eslint prettier eslint-config-prettier
```

### 4.3 フォーム関連ライブラリ（任意）

フォーム処理を効率化するためのライブラリ（必要に応じてインストール）：

```bash
# フォーム関連（MVPのステップ3以降で必要になる場合）
npm install react-hook-form zod @hookform/resolvers
```

## 5. Supabaseのセットアップ

### 5.1 Supabaseアカウントの作成

1. [Supabase公式サイト](https://supabase.com/)にアクセス
2. 「Start your project」をクリック
3. GitHubで新規登録またはログイン

### 5.2 新規プロジェクトの作成

1. ダッシュボードから「New Project」をクリック
2. 組織を選択（または新規作成）
3. 以下の情報を入力：
   - Name: `mycoffee-note`
   - Database Password: 安全なパスワードを設定（メモしておくこと）
   - Region: 最も近い地域を選択
4. 「Create new project」をクリック
5. プロジェクトが作成されるのを待つ（約1分）

### 5.3 データベーステーブルの作成

プロジェクトが作成されたら、SQL Editorを使用してテーブルを作成します：

1. 左側メニューの「SQL Editor」をクリック
2. 「New Query」をクリック
3. 以下のSQLを入力：

```sql
-- Usersテーブル
CREATE TABLE users (
  id UUID REFERENCES auth.users NOT NULL PRIMARY KEY,
  email TEXT NOT NULL,
  display_name TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- RLSポリシー
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分のプロフィールを参照可能" ON users
  FOR SELECT USING (auth.uid() = id);
CREATE POLICY "ユーザーは自分のプロフィールを編集可能" ON users
  FOR UPDATE USING (auth.uid() = id);

-- Recipesテーブル
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

-- RLSポリシー
ALTER TABLE recipes ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分のレシピを操作可能" ON recipes
  FOR ALL USING (auth.uid() = user_id);

-- BrewingParamsテーブル
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

-- RLSポリシー
ALTER TABLE brewing_params ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分のパラメータを操作可能" ON brewing_params
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM recipes 
      WHERE recipes.id = brewing_params.recipe_id 
      AND recipes.user_id = auth.uid()
    )
  );

-- Evaluationsテーブル
CREATE TABLE evaluations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  recipe_id UUID REFERENCES recipes(id) NOT NULL,
  user_id UUID REFERENCES users(id) NOT NULL,
  rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- RLSポリシー
ALTER TABLE evaluations ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分の評価を操作可能" ON evaluations
  FOR ALL USING (auth.uid() = user_id);
```

4. 「Run」をクリックして実行

### 5.4 認証設定

1. 左側メニューの「Authentication」→「Providers」をクリック
2. Email Providerが有効になっていることを確認
3. 「Save」をクリック

### 5.5 APIキーの取得

1. 左側メニューの「Project Settings」→「API」をクリック
2. 以下の情報をメモ：
   - Project URL
   - anon public (API Key)

## 6. プロジェクト設定

### 6.1 環境変数の設定

プロジェクトルートに`.env.local`ファイルを作成し、以下の内容を追加します：

```
VITE_SUPABASE_URL=あなたのプロジェクトURL
VITE_SUPABASE_ANON_KEY=あなたのanon public API Key
```

※上記の値は、Supabaseの「Project Settings」→「API」からコピーしたものに置き換えてください。

### 6.2 プロジェクト構造の作成

以下のフォルダ構造を作成します：

```bash
mkdir -p src/{components,pages,hooks,lib,types,assets}
mkdir -p src/components/{auth,common,layout,recipe}
mkdir -p src/pages/{auth,recipes}
mkdir -p src/hooks/{auth,form,data}
```

### 6.3 Supabaseクライアントの設定

`src/lib/supabase.ts`ファイルを作成し、以下の内容を追加します：

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 6.4 基本的な型定義

`src/types/database.ts`ファイルを作成し、以下の内容を追加します：

```typescript
// ユーザー
export interface User {
  id: string;
  email: string;
  display_name?: string;
  created_at: string;
  updated_at: string;
}

// レシピ
export interface Recipe {
  id: string;
  user_id: string;
  name: string;
  coffee_name: string;
  is_favorite: boolean;
  created_at: string;
  updated_at: string;
  origin?: string;
  roast_level?: string;
  notes?: string;
}

// 抽出方法の種類
export type BrewingMethod = 
  | 'drip' 
  | 'espresso' 
  | 'french_press' 
  | 'aeropress' 
  | 'moka_pot' 
  | 'cold_brew';

// 抽出パラメータ
export interface BrewingParams {
  id: string;
  recipe_id: string;
  brewing_method: BrewingMethod;
  coffee_amount: number;
  water_amount: number;
  brewing_time: number;
  water_temperature?: number;
  grind_size?: string;
  method_specific_params: Record<string, any>;
}

// 評価
export interface Evaluation {
  id: string;
  recipe_id: string;
  user_id: string;
  rating: number;
  notes?: string;
  created_at: string;
}
```

### 6.5 ESLintとPrettierの設定

`.eslintrc.cjs`を編集し、以下の内容に更新します：

```javascript
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],
  },
}
```

`.prettierrc`ファイルを作成し、以下の内容を追加します：

```json
{
  "semi": true,
  "tabWidth": 2,
  "printWidth": 100,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

## 7. Vercelへのデプロイ設定

### 7.1 Vercelアカウントの作成

1. [Vercel公式サイト](https://vercel.com/)にアクセス
2. GitHubでサインアップまたはログイン

### 7.2 プロジェクトのインポート

1. Vercelダッシュボードから「New Project」をクリック
2. GitHubリポジトリからmycoffee-noteを選択
3. 「Import」をクリック

### 7.3 環境変数の設定

1. 「Environment Variables」セクションに移動
2. 以下の環境変数を追加：
   - `VITE_SUPABASE_URL`：Supabaseプロジェクトの URL
   - `VITE_SUPABASE_ANON_KEY`：Supabaseプロジェクトの anon public API Key

### 7.4 デプロイ設定の確認と実行

1. プロジェクト設定を確認
2. フレームワークプリセットが「Vite」になっているか確認
3. 「Deploy」をクリック

### 7.5 Supabaseの認証リダイレクト設定

デプロイが完了したら：

1. Supabaseダッシュボードの「Authentication」→「Settings」→「URL Configuration」に移動
2. 「Site URL」にVercelのデプロイURL（例：https://mycoffee-note.vercel.app）を追加
3. 「Additional redirect URLs」に以下を追加：
   - `https://mycoffee-note.vercel.app/auth/callback`
   - `https://mycoffee-note.vercel.app/login`
4. 「Save」をクリック

## 8. トラブルシューティング

### 8.1 Viteのインストールエラー

**問題**: `npm create vite@latest` コマンドが失敗する

**解決策**:
1. Nodeのバージョンを確認（16.x以上が必要）
2. npm自体を更新: `npm install -g npm@latest`
3. 代替コマンド: `npx create-vite@latest . --template react-ts`

### 8.2 Supabase接続エラー

**問題**: Supabaseへの接続エラーが発生する

**解決策**:
1. `.env.local`の環境変数が正しく設定されているか確認
2. Supabaseプロジェクトが正常に起動しているか確認
3. ブラウザのコンソールで詳細なエラーメッセージを確認

### 8.3 TypeScriptのエラー

**問題**: TypeScriptのコンパイルエラーが発生する

**解決策**:
1. 依存関係が正しくインストールされているか確認: `npm install`
2. エディタを再起動
3. 具体的なエラーメッセージを調査・修正

### 8.4 GitでPushできない

**問題**: リポジトリへのPushに失敗する

**解決策**:
1. GitHubの認証情報を確認
2. リモートURLが正しいか確認: `git remote -v`
3. 必要に応じて認証の再設定: `git config --global user.name "あなたの名前"` と `git config --global user.email "あなたのメール"`

## 開発環境起動

セットアップが完了したら、開発サーバーを起動します：

```bash
npm run dev
```

ブラウザで `http://localhost:5173` にアクセスし、Viteの初期画面が表示されることを確認してください。

これで MyCoffee Note の開発環境のセットアップは完了です！開発ガイドに沿って実装を進めてください。

---

セットアップ中に問題が発生した場合は、エラーメッセージを確認し、公式ドキュメントや Stack Overflow で解決策を探してみてください。また、チーム内で知識を共有し、互いに助け合うことも大切です。
