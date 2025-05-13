# MyCoffee Note 初心者向け実装ガイド (第1部: 環境設定とプロジェクト初期化)

このガイドは、MyCoffee Note Web版MVPの開発を始める初心者開発者向けに、環境セットアップから最初のコンポーネント実装までをステップバイステップで説明します。

## 目次

1. [開発環境のセットアップ](#1-開発環境のセットアップ)
2. [プロジェクト初期設定](#2-プロジェクト初期設定)
3. [Supabaseプロジェクトのセットアップ](#3-supabaseプロジェクトのセットアップ)

## 1. 開発環境のセットアップ

### 1.1 必要なソフトウェア

以下のソフトウェアをインストールしてください：

- **Node.js (v16.x以上)**: [公式サイト](https://nodejs.org/)からダウンロード
- **VSCode**: [公式サイト](https://code.visualstudio.com/)からダウンロード
- **Git**: [公式サイト](https://git-scm.com/downloads)からダウンロード

### 1.2 推奨VSCode拡張機能

VSCodeで以下の拡張機能をインストールしてください：

- **ESLint**: JavaScript/TypeScriptのリンター
- **Prettier - Code formatter**: コードフォーマッター
- **ES7+ React/Redux/React-Native snippets**: Reactのコードスニペット
- **TypeScript Error Translator**: TypeScriptエラーを分かりやすく表示
- **GitLens**: Git操作の拡張

### 1.3 GitHub設定

1. GitHubアカウントを持っていない場合は、[GitHub](https://github.com/)でアカウントを作成
2. リポジトリを作成：
   - GitHubにログイン
   - 「New repository」をクリック
   - リポジトリ名に「mycoffee-note」を入力
   - 「Initialize this repository with a README」にチェック
   - 「Create repository」をクリック

## 2. プロジェクト初期設定

### 2.1 プロジェクトのクローンとセットアップ

```bash
# リポジトリをクローン（自分のGitHubユーザー名に置き換えてください）
git clone https://github.com/あなたのユーザー名/mycoffee-note.git
cd mycoffee-note

# Viteプロジェクトを作成
npm create vite@latest . -- --template react-ts

# 依存関係をインストール
npm install
```

### 2.2 追加の依存関係をインストール

```bash
# ルーティングとSupabase
npm install react-router-dom @supabase/supabase-js

# 日付操作
npm install date-fns

# フォーム操作
npm install react-hook-form zod @hookform/resolvers
```

### 2.3 プロジェクト構造を作成

以下のフォルダ構造を作成してください：

```bash
mkdir -p src/{components,pages,hooks,lib,types,assets}
mkdir -p src/components/{auth,common,layout,recipe}
mkdir -p src/pages/{auth,recipes}
mkdir -p src/hooks/{auth,form,data}
```

### 2.4 基本設定ファイルの作成

#### .gitignore ファイルを更新

```
# .gitignore
node_modules
dist
.env
.env.local
.DS_Store
*.log
```

#### ESLintとPrettierの設定

```bash
# ESLintとPrettierをインストール
npm install -D eslint prettier eslint-config-prettier
```

`.eslintrc.cjs`ファイルを編集：

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

`.prettierrc`ファイルを作成：

```json
{
  "semi": true,
  "tabWidth": 2,
  "printWidth": 100,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

### 2.5 環境変数の設定

`.env.local`ファイルを作成（※このファイルはGitに含めないでください）：

```
VITE_SUPABASE_URL=あとで追加します
VITE_SUPABASE_ANON_KEY=あとで追加します
```

## 3. Supabaseプロジェクトのセットアップ

### 3.1 Supabaseアカウント作成とプロジェクト設定

1. [Supabase](https://supabase.com/)にアクセスし、GitHub連携でサインアップ
2. 「New Project」をクリックして新しいプロジェクトを作成
3. プロジェクト名に「mycoffee-note」を入力
4. パスワードを設定（後で使用するので保存してください）
5. リージョンを選択（地理的に最も近いものを選ぶとよいでしょう）
6. 「Create new project」をクリック

### 3.2 .env.localファイルの更新

プロジェクト作成後、「Project Settings」→「API」から以下の情報を取得し、`.env.local`を更新：

```
VITE_SUPABASE_URL=あなたのプロジェクトURL
VITE_SUPABASE_ANON_KEY=あなたのプロジェクトAnon Key
```

### 3.3 Supabaseクライアントの設定

`src/lib/supabase.ts`ファイルを作成：

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 3.4 データベーステーブルの作成

SupabaseのSQL Editorで以下のSQLを実行：

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

### 3.5 認証設定

1. Supabaseダッシュボードの「Authentication」→「Providers」を開く
2. 「Email」が有効になっていることを確認
3. 「Site URL」に`http://localhost:5173`を追加
4. 「Additional redirect URLs」に`http://localhost:5173/auth/callback`を追加
5. 「Save」をクリック

---

次の第2部では、基本コンポーネントと認証機能の実装について説明します。
