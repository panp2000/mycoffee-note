# MyCoffee Note 簡略化データモデル仕様書

## 1. 概要

このドキュメントは、MyCoffee Note MVPのデータベース設計を初心者開発者向けに簡略化して説明したものです。最小限の複雑さで必要な機能を実現できるデータモデルを定義しています。

## 2. エンティティ関連図 (ER図)

```
┌────────────┐       ┌────────────┐       ┌────────────┐
│    User    │       │   Recipe   │       │BrewingParams│
├────────────┤       ├────────────┤       ├────────────┤
│id (PK)     │       │id (PK)     │       │id (PK)     │
│email       │<──────│user_id (FK)│       │recipe_id(FK)│──────┐
│display_name│       │name        │       │brewing_method│      │
│created_at  │       │coffee_name │       │coffee_amount │      │
│updated_at  │       │is_favorite │       │water_amount  │      │
└────────────┘       │created_at  │       │brewing_time  │      │
                     │updated_at  │       │water_temp    │      │
                     │origin      │       │grind_size    │      │
                     │roast_level │       │method_params │      │
                     │notes       │       └────────────┘       │
                     └────────────┘                           │
                           │                                  │
                           │       ┌────────────┐             │
                           └───────│ Evaluation │             │
                                   ├────────────┤             │
                                   │id (PK)     │             │
                                   │recipe_id(FK)├─────────────┘
                                   │user_id (FK)│
                                   │rating      │
                                   │notes       │
                                   │created_at  │
                                   └────────────┘
```

## 3. テーブル定義（簡略版）

### 3.1 Userテーブル

Supabaseの認証システムと連携するユーザー情報を格納します。

| カラム名 | データ型 | 説明 | 
|---------|---------|------|
| id | UUID | プライマリキー（Supabase Auth UUIDと一致） |
| email | TEXT | メールアドレス |
| display_name | TEXT | 表示名（オプション） |
| created_at | TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | 更新日時 |

### 3.2 Recipeテーブル

コーヒーレシピの基本情報を格納します。

| カラム名 | データ型 | 説明 | 
|---------|---------|------|
| id | UUID | プライマリキー |
| user_id | UUID | 作成者ID（Userテーブルの外部キー） |
| name | TEXT | レシピ名 |
| coffee_name | TEXT | コーヒー豆名 |
| is_favorite | BOOLEAN | お気に入りフラグ |
| created_at | TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | 更新日時 |
| origin | TEXT | 産地（オプション） |
| roast_level | TEXT | 焙煎度合い（オプション） |
| notes | TEXT | メモ（オプション） |

### 3.3 BrewingParamsテーブル

抽出パラメータ情報を格納します。

| カラム名 | データ型 | 説明 | 
|---------|---------|------|
| id | UUID | プライマリキー |
| recipe_id | UUID | レシピID（Recipeテーブルの外部キー） |
| brewing_method | TEXT | 抽出方法（例: ドリップ、エスプレッソ） |
| coffee_amount | NUMERIC | コーヒー豆の量（g） |
| water_amount | NUMERIC | 湯量/水量（ml） |
| brewing_time | INTEGER | 抽出時間（秒） |
| water_temperature | INTEGER | 湯温（℃）（オプション） |
| grind_size | TEXT | 挽き目（オプション） |
| method_specific_params | JSONB | 抽出方法別の特殊パラメータ（JSONで保存） |

### 3.4 Evaluationテーブル

レシピの評価情報を格納します。

| カラム名 | データ型 | 説明 | 
|---------|---------|------|
| id | UUID | プライマリキー |
| recipe_id | UUID | レシピID（Recipeテーブルの外部キー） |
| user_id | UUID | 評価者ID（Userテーブルの外部キー） |
| rating | INTEGER | 星評価（1-5） |
| notes | TEXT | 評価メモ（オプション） |
| created_at | TIMESTAMP | 評価日時 |

## 4. JSONBフィールドの構造

### 4.1 method_specific_params (BrewingParams)

抽出方法ごとの特殊なパラメータをJSONB形式で格納します。MVPでは以下の6つの抽出方法に対応します。

#### ドリップ（Drip）の例:

```json
{
  "dripper_type": "ハリオV60",
  "filter_type": "ペーパー",
  "bloom_time": 30,
  "pour_count": 3
}
```

#### エスプレッソ（Espresso）の例:

```json
{
  "pressure": 9,
  "shot_weight": 36,
  "pre_infusion_time": 5
}
```

#### フレンチプレス（FrenchPress）の例:

```json
{
  "steep_time": 240,
  "stir": true
}
```

#### エアロプレス（AeroPress）の例:

```json
{
  "steep_time": 60,
  "inverted": true
}
```

#### モカポット（MokaPot）の例:

```json
{
  "heat_level": "medium",
  "fill_level": "valve"
}
```

#### 水出し（ColdBrew）の例:

```json
{
  "steep_time": 43200,
  "room_temp": true
}
```

## 5. データアクセスの基本パターン

### 5.1 レシピの取得（パラメータと一緒に）

```typescript
// src/lib/api/recipes.ts
export async function getRecipeWithParams(recipeId: string) {
  const { data, error } = await supabase
    .from('recipes')
    .select(`
      *,
      brewing_params:brewing_params(*)
    `)
    .eq('id', recipeId)
    .single();
  
  if (error) throw error;
  return data;
}
```

### 5.2 ユーザーのレシピ一覧取得

```typescript
// src/lib/api/recipes.ts
export async function getUserRecipes(userId: string) {
  const { data, error } = await supabase
    .from('recipes')
    .select('*')
    .eq('user_id', userId)
    .order('created_at', { ascending: false });
  
  if (error) throw error;
  return data;
}
```

### 5.3 新規レシピ作成（パラメータ含む）

```typescript
// src/lib/api/recipes.ts
export async function createRecipeWithParams(
  recipe: Omit<Recipe, 'id' | 'created_at' | 'updated_at'>,
  params: Omit<BrewingParams, 'id' | 'recipe_id'>
) {
  // トランザクション的な処理（Supabaseでは真のトランザクションはないが、エラー時には一貫性を保つ）
  const { data: recipeData, error: recipeError } = await supabase
    .from('recipes')
    .insert(recipe)
    .select()
    .single();
  
  if (recipeError) throw recipeError;
  
  const { error: paramsError } = await supabase
    .from('brewing_params')
    .insert({
      ...params,
      recipe_id: recipeData.id
    });
  
  if (paramsError) {
    // パラメータ作成に失敗した場合、レシピも削除して一貫性を保つ
    await supabase.from('recipes').delete().eq('id', recipeData.id);
    throw paramsError;
  }
  
  return recipeData;
}
```

## 6. タイプスクリプト型定義

```typescript
// src/types/database.ts

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

// 抽出方法の種類（型安全のための列挙型）
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
  method_specific_params: Record<string, any>; // 抽出方法別のパラメータ
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

// 抽出方法別のパラメータ型（オプション）
export interface DripParams {
  dripper_type?: string;
  filter_type?: string;
  bloom_time?: number;
  pour_count?: number;
}

export interface EspressoParams {
  pressure?: number;
  shot_weight?: number;
  pre_infusion_time?: number;
}

// 他の抽出方法も同様に定義...
```

## 7. Supabase RLSポリシー（セキュリティルール）

各テーブルに適用する行レベルセキュリティポリシーを定義します。これにより、ユーザーは自分のデータのみにアクセスできます。

### 7.1 Usersテーブル

```sql
-- ユーザーは自分のプロフィールのみ参照/編集可能
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "ユーザーは自分のプロフィールを参照可能" 
ON users FOR SELECT USING (auth.uid() = id);

CREATE POLICY "ユーザーは自分のプロフィールを編集可能" 
ON users FOR UPDATE USING (auth.uid() = id);
```

### 7.2 Recipesテーブル

```sql
-- ユーザーは自分のレシピのみ操作可能
ALTER TABLE recipes ENABLE ROW LEVEL SECURITY;

CREATE POLICY "ユーザーは自分のレシピを操作可能" 
ON recipes FOR ALL USING (auth.uid() = user_id);
```

### 7.3 BrewingParamsテーブル

```sql
-- ユーザーは自分のレシピに関連するパラメータのみ操作可能
ALTER TABLE brewing_params ENABLE ROW LEVEL SECURITY;

CREATE POLICY "ユーザーは自分のパラメータを操作可能" 
ON brewing_params FOR ALL USING (
  EXISTS (
    SELECT 1 FROM recipes 
    WHERE recipes.id = brewing_params.recipe_id 
    AND recipes.user_id = auth.uid()
  )
);
```

### 7.4 Evaluationテーブル

```sql
-- ユーザーは自分の評価のみ操作可能
ALTER TABLE evaluations ENABLE ROW LEVEL SECURITY;

CREATE POLICY "ユーザーは自分の評価を操作可能" 
ON evaluations FOR ALL USING (auth.uid() = user_id);
```

## 8. 初心者向けデータモデル実装のヒント

1. **シンプルから始める**: まずは基本的なCRUD操作を実装し、後から複雑な機能を追加しましょう。

2. **Supabaseの活用**: Supabaseは自動生成されたAPIを提供しているため、複雑なバックエンド開発なしでデータベース操作が可能です。

3. **RLSの重要性**: Row Level Security (RLS)を適切に設定することで、セキュリティを確保できます。

4. **JSONBフィールドの利点**: 抽出方法別パラメータなど、柔軟な構造のデータはJSONBフィールドを活用することで、スキーマの複雑さを軽減できます。

5. **型安全性の確保**: TypeScriptの型定義を使って、データの整合性を確保しましょう。

6. **関連データの取得**: JOINの代わりに、Supabaseの`select('*, related_table(*)')`構文を使って関連データを取得できます。

## 9. 将来の拡張性

このシンプルなデータモデルは、将来的に以下のように拡張可能です：

1. **詳細な評価**: 風味プロファイル詳細評価のためのカラム追加
2. **タグシステム**: レシピのタグ付け機能
3. **バージョニング**: レシピの変更履歴追跡
4. **共有機能**: 公開/非公開設定や共有URLのサポート
5. **コメント機能**: レシピへのコメント追加

MVPフェーズでは、これらの機能は優先しませんが、データモデルはこれらの拡張に対応できるように設計されています。
