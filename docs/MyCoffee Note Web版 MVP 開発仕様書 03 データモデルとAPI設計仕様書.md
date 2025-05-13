# MyCoffee Note Web版 MVP 技術仕様書

## 3. データモデルとAPI設計仕様書

### 3.1 コアエンティティモデル

#### User
```typescript
interface User {
  id: string;                // Supabase Auth UUIDと一致
  email: string;             // メールアドレス
  display_name?: string;     // 表示名
  avatar_url?: string;       // プロフィール画像URL
  created_at: Date;          // 作成日時
  updated_at: Date;          // 更新日時
  last_sign_in_at?: Date;    // 最終ログイン日時
  settings: UserSettings;    // ユーザー設定（JSONBで保存）
}

interface UserSettings {
  theme?: 'light' | 'dark' | 'system';  // テーマ設定
  measurement_unit?: 'metric' | 'imperial'; // 単位系設定
  notification_preferences?: {          // 通知設定
    email_notifications: boolean;
    // 将来的な通知設定オプション
  };
}
```

#### Recipe
```typescript
interface Recipe {
  id: string;              // 一意のID (UUID)
  user_id: string;         // 作成者ID (User.idへの参照)
  name: string;            // レシピ名
  coffee_name: string;     // コーヒー豆名
  is_public: boolean;      // 公開/非公開設定（デフォルトはfalse）
  is_favorite: boolean;    // お気に入りフラグ
  version: number;         // バージョン番号（楽観的ロック用）
  
  // メタデータ
  created_at: Date;        // 作成日時
  updated_at: Date;        // 更新日時
  
  // オプション情報
  origin?: string;         // 産地
  roast_level?: string;    // 焙煎度合い
  roast_date?: Date;       // 焙煎日
  expiry_date?: Date;      // 賞味期限
  purchase_location?: string; // 購入店/メーカー
  notes?: string;          // メモ
  tags?: string[];         // タグ（配列として保存）
}
```

#### BrewingParams
```typescript
interface BrewingParams {
  id: string;              // 一意のID
  recipe_id: string;       // 関連するレシピID
  version: number;         // バージョン番号
  
  // 基本パラメータ（全抽出方法共通）
  brewing_method: string;   // 抽出方法
  coffee_amount: number;    // 豆の量(g)
  water_amount: number;     // 湯量/水量(ml)
  brewing_time: number;     // 抽出時間(秒)
  water_temperature?: number; // 湯温(℃)
  grind_size?: string;      // 挽き目の粗さ
  
  // 抽出方法別のパラメータ（JSONB形式で保存）
  method_specific_params: Record<string, any>;
  
  // メタデータ
  created_at: Date;        // 作成日時
  updated_at: Date;        // 更新日時
}
```

#### Equipment
```typescript
interface Equipment {
  id: string;              // 一意のID
  user_id: string;         // 所有者ID
  type: string;            // 器具タイプ（ドリッパー、グラインダー等）
  category?: string;       // カテゴリー（円錐型、平底型等）
  manufacturer?: string;   // メーカー名
  model?: string;          // 商品名/型番
  notes?: string;          // 備考
  version: number;         // バージョン番号
  
  // メタデータ
  created_at: Date;        // 作成日時
  updated_at: Date;        // 更新日時
}
```

#### RecipeEquipment (中間テーブル)
```typescript
interface RecipeEquipment {
  recipe_id: string;       // レシピID
  equipment_id: string;    // 器具ID
  settings?: Record<string, any>; // 器具の設定情報（JSONB）
}
```

#### Evaluation
```typescript
interface Evaluation {
  id: string;              // 一意のID
  recipe_id: string;       // 関連するレシピID
  user_id: string;         // 評価者ID
  overall_rating: number;  // 総合評価（1-5）
  version: number;         // バージョン番号
  
  // 風味バランス（任意）
  taste_balance?: {
    acidity?: number;      // 酸味（1-5）
    bitterness?: number;   // 苦味（1-5）
    sweetness?: number;    // 甘み（1-5）
    body?: number;         // コク（1-5）
    aftertaste?: number;   // 後味（1-5）
  };
  
  // お気に入り度
  favorite_level?: 'yes' | 'neutral' | 'no'; // また飲みたいか
  
  // 風味タグ（任意、複数選択可）
  flavor_tags?: string[];
  
  // テイスティングノート（自由記述）
  tasting_notes?: string;
  
  // 改善メモ（次回の改善点）
  improvement_notes?: string;
  
  // メタデータ
  created_at: Date;         // 評価日時
  updated_at: Date;         // 更新日時
}
```

### 3.2 PostgreSQLテーブル設計

```sql
-- Usersテーブル (Supabase Authと連携)
CREATE TABLE users (
  id UUID REFERENCES auth.users NOT NULL PRIMARY KEY,
  email TEXT NOT NULL,
  display_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_sign_in_at TIMESTAMPTZ,
  settings JSONB DEFAULT '{}'::jsonb
);

-- RLSポリシー: 自分のプロフィールのみ編集可能
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
  is_public BOOLEAN NOT NULL DEFAULT false,
  is_favorite BOOLEAN NOT NULL DEFAULT false,
  version INTEGER NOT NULL DEFAULT 1,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  origin TEXT,
  roast_level TEXT,
  roast_date DATE,
  expiry_date DATE,
  purchase_location TEXT,
  notes TEXT,
  tags TEXT[] DEFAULT '{}'::TEXT[]
);

-- バージョン自動インクリメントトリガー
CREATE FUNCTION increment_version() RETURNS TRIGGER AS $$
BEGIN
    NEW.version := OLD.version + 1;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER recipes_version_increment
BEFORE UPDATE ON recipes
FOR EACH ROW
EXECUTE FUNCTION increment_version();

-- RLSポリシー: 自分のレシピを編集、公開レシピを閲覧可能
ALTER TABLE recipes ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分のレシピを操作可能" ON recipes
  FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "ユーザーは公開レシピを閲覧可能" ON recipes
  FOR SELECT USING (is_public = true);

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
  version INTEGER NOT NULL DEFAULT 1,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- バージョン自動インクリメントトリガー
CREATE TRIGGER brewing_params_version_increment
BEFORE UPDATE ON brewing_params
FOR EACH ROW
EXECUTE FUNCTION increment_version();

-- RLSポリシー: レシピと同じアクセス制御
ALTER TABLE brewing_params ENABLE ROW LEVEL SECURITY;
CREATE POLICY "ユーザーは自分のパラメータを操作可能" ON brewing_params
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM recipes 
      WHERE recipes.id = brewing_params.recipe_id 
      AND recipes.user_id = auth.uid()
    )
  );
CREATE POLICY "ユーザーは公開レシピのパラメータを閲覧可能" ON brewing_params
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM recipes 
      WHERE recipes.id = brewing_params.recipe_id 
      AND recipes.is_public = true
    )
  );

-- 他のテーブルも同様に定義
```

### 3.3 データアクセスアプローチ

MVPフェーズでは、コンポーネントから直接Supabaseクライアントを使用し、TanStack Queryのキャッシュ機能を活用したシンプルなアプローチを採用します。

#### シンプルなカスタムフック例

```typescript
// シンプルなカスタムフック例
export function useRecipes(filters?: RecipeFilters) {
  return useQuery({
    queryKey: ['recipes', filters],
    queryFn: async () => {
      // 直接Supabaseクライアントを使用
      let query = supabase
        .from('recipes')
        .select('*, brewing_params(*)')
        .order('updated_at', { ascending: false });
      
      // 必要に応じてフィルター適用
      if (filters?.brewing_method) {
        query = query.eq('brewing_params.brewing_method', filters.brewing_method);
      }
      
      const { data, error } = await query;
      if (error) throw error;
      return data;
    },
  });
}

// 単純なミューテーション例
export function useCreateRecipe() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async ({ recipe, brewingParams }: RecipeWithParams) => {
      // トランザクション的な処理を単一の関数内で実行
      const { data, error } = await supabase
        .from('recipes')
        .insert(recipe)
        .select('id')
        .single();
      
      if (error) throw error;
      
      const { error: paramsError } = await supabase
        .from('brewing_params')
        .insert({ ...brewingParams, recipe_id: data.id });
      
      if (paramsError) throw paramsError;
      
      return data.id;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['recipes'] });
    }
  });
}
```

#### 楽観的ロック実装例

```typescript
// 楽観的ロックの実装例
async function updateRecipe(id: string, updates: Partial<Recipe>, currentVersion: number) {
  const { data, error } = await supabase
    .from('recipes')
    .update({ ...updates, version: currentVersion + 1 })
    .eq('id', id)
    .eq('version', currentVersion)
    .select()
    .single();
  
  if (error) {
    // バージョン競合処理
    if (error.code === '23514') {
      throw new Error('このレシピは他の場所で更新されています。最新版を取得してください。');
    }
    throw error;
  }
  
  return data;
}
```

### 3.4 抽出方法別パラメータ定義

抽出方法別の特殊パラメータは、`BrewingParams.method_specific_params`にJSONB形式で保存します：

```typescript
// ドリップの場合
interface DripParams {
  dripper?: {              // ドリッパー情報
    type?: string;         // 種類（円錐/波型/平底等）
    material?: string;     // 材質（セラミック/プラスチック/金属等）
  };
  filter?: {               // フィルター情報
    type?: string;         // 種類（ペーパー/金属/クロス等）
    brand?: string;        // メーカー・ブランド
  };
  bloom_time?: number;      // 蒸らし時間（秒）
  pour_patterns?: {         // 注湯パターン
    count?: number;        // 注湯回数
    intervals?: number[];  // 各注湯の間隔（秒）
    amounts?: number[];    // 各注湯の量（ml）
  };
}

// エスプレッソの場合
interface EspressoParams {
  machine?: string;        // マシン情報
  portafilter?: string;    // ポルタフィルタータイプ
  basket?: {               // バスケット情報
    type?: string;         // 種類（標準/VST/IMS等）
    size?: number;         // サイズ（g）
  };
  pressure?: number;       // 抽出圧力（bar）
  shot_weight?: number;     // 抽出量（g）
  pre_infusion?: {          // プレインフュージョン
    enabled: boolean;      // 有無
    time?: number;         // 時間（秒）
    pressure?: number;     // 圧力（bar）
  };
}

// フレンチプレスの場合
interface FrenchPressParams {
  steep_time: number;       // 浸漬時間（秒）
  stirring?: {             // かき混ぜ
    enabled: boolean;      // 有無
    times?: number;        // 回数
    timing?: number[];     // タイミング（秒）
  };
  press_speed?: 'slow' | 'medium' | 'fast'; // プレス速度
}

// 他の抽出方法も同様に定義...
```

### 3.5 データバージョニングとスキーマ進化戦略

#### MVPフェーズのアプローチ
1. **既存のタイムスタンプの活用**:
   - Supabaseが自動提供する`created_at`と`updated_at`を最大限活用
   - これらは変更の時系列追跡に十分な情報を提供

2. **バージョニングの基本実装**:
   - 更新時に`version`を条件に含めることで、基本的な競合検出が可能
   - 各主要テーブルに`version`列と自動インクリメントトリガーを実装

3. **将来の拡張への備え**:
   - テーブル設計時に列の追加が容易なスキーマ設計
   - JSONBフィールドを活用し、スキーマ変更を最小限に抑える

#### 将来検討すべきバージョニング拡張

1. **変更履歴テーブル**:
   複雑な監査要件やロールバック機能が必要になった場合の拡張として
```sql
CREATE TABLE recipe_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  recipe_id UUID REFERENCES recipes(id),
  changed_by UUID REFERENCES users(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  version INTEGER NOT NULL,
  data JSONB NOT NULL -- そのバージョンでのレシピ全データのスナップショット
);
```

### 3.6 将来的な拡張性

#### 拡張を見据えたデータモデルの工夫

1. **JSONBフィールドの活用**:
   - `method_specific_params`など可変パラメータはJSONBで保存し、スキーマ変更なしで拡張可能に
   - `user_settings`に新しい設定項目を追加可能

2. **一般的なタグシステム**:
   - 現状は単純な配列として`tags`を実装
   - 将来的に独立した`tags`テーブルへの移行も容易に

3. **コミュニティ機能への拡張準備**:
   - `is_public`フラグはすでに実装
   - 将来的に`shared_with`や権限管理システムへの拡張が可能

4. **データ検証と制約**:
   - PostgreSQLの制約（CHECK制約など）を活用
   - アプリケーション層でのバリデーション（Zod）との二重検証