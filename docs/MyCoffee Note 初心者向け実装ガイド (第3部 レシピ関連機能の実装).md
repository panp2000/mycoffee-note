# MyCoffee Note 初心者向け実装ガイド (第3部: レシピ関連機能の実装)

## 目次

1. [レシピデータアクセス関数の作成](#1-レシピデータアクセス関数の作成)
2. [レシピフォームの作成](#2-レシピフォームの作成)
3. [レシピリストコンポーネントの作成](#3-レシピリストコンポーネントの作成)
4. [レシピページの更新](#4-レシピページの更新)

## 1. レシピデータアクセス関数の作成

`src/lib/api/recipes.ts`ファイルを作成：

```typescript
import { supabase } from '../supabase';
import { Recipe, BrewingParams } from '../../types/database';

// レシピ一覧を取得する
export async function getRecipes() {
  const { data, error } = await supabase
    .from('recipes')
    .select('*')
    .order('created_at', { ascending: false });

  if (error) throw error;
  return data || [];
}

// レシピ詳細を取得する（抽出パラメータ含む）
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

// 新規レシピを作成する
export async function createRecipe(
  recipe: Omit<Recipe, 'id' | 'created_at' | 'updated_at'>,
  params: Omit<BrewingParams, 'id' | 'recipe_id' | 'created_at'>
) {
  // レシピを作成
  const { data: recipeData, error: recipeError } = await supabase
    .from('recipes')
    .insert(recipe)
    .select()
    .single();

  if (recipeError) throw recipeError;

  // 抽出パラメータを作成
  const { error: paramsError } = await supabase
    .from('brewing_params')
    .insert({
      ...params,
      recipe_id: recipeData.id
    });

  if (paramsError) {
    // パラメータ作成に失敗した場合、レシピも削除
    await supabase.from('recipes').delete().eq('id', recipeData.id);
    throw paramsError;
  }

  return recipeData;
}

// レシピを更新する
export async function updateRecipe(
  recipeId: string,
  recipe: Partial<Omit<Recipe, 'id' | 'created_at' | 'updated_at'>>
) {
  const { data, error } = await supabase
    .from('recipes')
    .update(recipe)
    .eq('id', recipeId)
    .select()
    .single();

  if (error) throw error;
  return data;
}

// レシピを削除する
export async function deleteRecipe(recipeId: string) {
  // 関連する抽出パラメータも削除される（外部キー制約のため）
  const { error } = await supabase
    .from('recipes')
    .delete()
    .eq('id', recipeId);

  if (error) throw error;
  return true;
}

// お気に入り状態を切り替える
export async function toggleFavorite(recipeId: string, isFavorite: boolean) {
  const { data, error } = await supabase
    .from('recipes')
    .update({ is_favorite: isFavorite })
    .eq('id', recipeId)
    .select()
    .single();

  if (error) throw error;
  return data;
}
```

## 2. レシピフォームの作成

`src/components/recipe/RecipeForm.tsx`を作成：

```tsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import { createRecipe } from '../../lib/api/recipes';
import { BrewingMethod } from '../../types/database';
import './RecipeForm.css';

// 抽出方法の選択肢
const brewingMethods = [
  { value: 'drip', label: 'ドリップ' },
  { value: 'espresso', label: 'エスプレッソ' },
  { value: 'french_press', label: 'フレンチプレス' },
  { value: 'aeropress', label: 'エアロプレス' },
  { value: 'moka_pot', label: 'モカポット' },
  { value: 'cold_brew', label: '水出し' },
];

export default function RecipeForm() {
  const navigate = useNavigate();
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // フォーム状態
  const [name, setName] = useState('');
  const [coffeeName, setCoffeeName] = useState('');
  const [origin, setOrigin] = useState('');
  const [roastLevel, setRoastLevel] = useState('');
  const [notes, setNotes] = useState('');
  const [brewingMethod, setBrewingMethod] = useState<BrewingMethod>('drip');
  const [coffeeAmount, setCoffeeAmount] = useState(18);
  const [waterAmount, setWaterAmount] = useState(300);
  const [brewingTime, setBrewingTime] = useState(180); // 秒
  const [waterTemperature, setWaterTemperature] = useState(93);
  const [grindSize, setGrindSize] = useState('中細挽き');
  
  // 抽出方法別のパラメータ（ドリップの例）
  const [dripperType, setDripperType] = useState('V60');
  const [filterType, setFilterType] = useState('ペーパー');
  const [bloomTime, setBloomTime] = useState(30);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!user) {
      setError('ログインが必要です');
      return;
    }
    
    setLoading(true);
    setError(null);
    
    try {
      // 抽出方法別のパラメータを設定
      let methodSpecificParams: Record<string, any> = {};
      
      if (brewingMethod === 'drip') {
        methodSpecificParams = {
          dripper_type: dripperType,
          filter_type: filterType,
          bloom_time: bloomTime
        };
      }
      // 他の抽出方法も同様に条件分岐で実装
      
      // レシピを作成
      await createRecipe(
        {
          user_id: user.id,
          name,
          coffee_name: coffeeName,
          is_favorite: false,
          origin,
          roast_level: roastLevel,
          notes
        },
        {
          brewing_method: brewingMethod,
          coffee_amount: coffeeAmount,
          water_amount: waterAmount,
          brewing_time: brewingTime,
          water_temperature: waterTemperature,
          grind_size: grindSize,
          method_specific_params: methodSpecificParams
        }
      );
      
      // 成功したらレシピ一覧に戻る
      navigate('/recipes');
    } catch (err: any) {
      setError(err.message || 'レシピの作成に失敗しました');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="recipe-form-container">
      <h1>新規レシピ作成</h1>
      
      {error && <div className="error-message">{error}</div>}
      
      <form onSubmit={handleSubmit}>
        <div className="form-section">
          <h2>基本情報</h2>
          
          <div className="form-group">
            <label htmlFor="name">レシピ名 *</label>
            <input 
              id="name"
              type="text"
              value={name}
              onChange={(e) => setName(e.target.value)}
              required
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="coffeeName">コーヒー豆名 *</label>
            <input 
              id="coffeeName"
              type="text"
              value={coffeeName}
              onChange={(e) => setCoffeeName(e.target.value)}
              required
            />
          </div>
          
          <div className="form-row">
            <div className="form-group">
              <label htmlFor="origin">産地</label>
              <input 
                id="origin"
                type="text"
                value={origin}
                onChange={(e) => setOrigin(e.target.value)}
              />
            </div>
            
            <div className="form-group">
              <label htmlFor="roastLevel">焙煎度合い</label>
              <select
                id="roastLevel"
                value={roastLevel}
                onChange={(e) => setRoastLevel(e.target.value)}
              >
                <option value="">選択してください</option>
                <option value="ライトロースト">ライトロースト</option>
                <option value="ミディアムロースト">ミディアムロースト</option>
                <option value="ミディアムダークロースト">ミディアムダークロースト</option>
                <option value="ダークロースト">ダークロースト</option>
              </select>
            </div>
          </div>
        </div>
        
        <div className="form-section">
          <h2>抽出パラメータ</h2>
          
          <div className="form-group">
            <label htmlFor="brewingMethod">抽出方法 *</label>
            <select
              id="brewingMethod"
              value={brewingMethod}
              onChange={(e) => setBrewingMethod(e.target.value as BrewingMethod)}
              required
            >
              {brewingMethods.map((method) => (
                <option key={method.value} value={method.value}>
                  {method.label}
                </option>
              ))}
            </select>
          </div>
          
          <div className="form-row">
            <div className="form-group">
              <label htmlFor="coffeeAmount">豆の量 (g) *</label>
              <input 
                id="coffeeAmount"
                type="number"
                min="1"
                step="0.1"
                value={coffeeAmount}
                onChange={(e) => setCoffeeAmount(parseFloat(e.target.value))}
                required
              />
            </div>
            
            <div className="form-group">
              <label htmlFor="waterAmount">湯量 (ml) *</label>
              <input 
                id="waterAmount"
                type="number"
                min="1"
                value={waterAmount}
                onChange={(e) => setWaterAmount(parseInt(e.target.value))}
                required
              />
            </div>
          </div>
          
          {/* 他の入力フィールド... */}
          
          {/* 抽出方法別のパラメータ（ドリップの例） */}
          {brewingMethod === 'drip' && (
            <div className="method-specific-params">
              <h3>ドリップパラメータ</h3>
              
              {/* ドリップ固有のパラメータフィールド... */}
            </div>
          )}
        </div>
        
        <div className="form-section">
          <h2>メモ</h2>
          
          <div className="form-group">
            <textarea
              value={notes}
              onChange={(e) => setNotes(e.target.value)}
              rows={4}
              placeholder="抽出のコツや気づいたことを自由に記録しましょう。"
            ></textarea>
          </div>
        </div>
        
        <div className="form-actions">
          <button type="button" className="cancel-button" onClick={() => navigate('/recipes')}>
            キャンセル
          </button>
          <button type="submit" className="submit-button" disabled={loading}>
            {loading ? '保存中...' : 'レシピを保存'}
          </button>
        </div>
      </form>
    </div>
  );
}
```

`src/components/recipe/RecipeForm.css`を作成（基本的なスタイル）:

```css
.recipe-form-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}

.form-section {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

/* 他のスタイル定義... */
```

## 3. レシピリストコンポーネントの作成

`src/components/recipe/RecipeList.tsx`を作成：

```tsx
import { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { getRecipes, toggleFavorite } from '../../lib/api/recipes';
import { Recipe } from '../../types/database';
import './RecipeList.css';

export default function RecipeList() {
  const [recipes, setRecipes] = useState<Recipe[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function loadRecipes() {
      try {
        const data = await getRecipes();
        setRecipes(data);
      } catch (err: any) {
        setError(err.message || 'レシピの取得に失敗しました');
      } finally {
        setLoading(false);
      }
    }

    loadRecipes();
  }, []);

  const handleToggleFavorite = async (recipeId: string, isFavorite: boolean) => {
    try {
      await toggleFavorite(recipeId, !isFavorite);
      
      // 状態を更新
      setRecipes((prevRecipes) =>
        prevRecipes.map((recipe) =>
          recipe.id === recipeId
            ? { ...recipe, is_favorite: !isFavorite }
            : recipe
        )
      );
    } catch (err: any) {
      console.error('お気に入り切り替えエラー:', err);
    }
  };

  if (loading) {
    return <div className="loading">読み込み中...</div>;
  }

  if (error) {
    return <div className="error-message">{error}</div>;
  }

  if (recipes.length === 0) {
    return (
      <div className="recipes-empty">
        <div className="empty-message">
          <h2>レシピがまだありません</h2>
          <p>「新規レシピ作成」ボタンをクリックして、最初のレシピを作成しましょう。</p>
          <Link to="/recipes/new" className="create-button large">
            最初のレシピを作成
          </Link>
        </div>
      </div>
    );
  }

  return (
    <div className="recipe-list">
      {recipes.map((recipe) => (
        <div key={recipe.id} className="recipe-card">
          <div className="recipe-card-header">
            <h2>{recipe.name}</h2>
            <button
              className={`favorite-button ${recipe.is_favorite ? 'active' : ''}`}
              onClick={() => handleToggleFavorite(recipe.id, recipe.is_favorite)}
              aria-label={recipe.is_favorite ? 'お気に入りから削除' : 'お気に入りに追加'}
            >
              {recipe.is_favorite ? '★' : '☆'}
            </button>
          </div>
          
          <div className="recipe-card-content">
            <p className="coffee-name">{recipe.coffee_name}</p>
            
            {recipe.origin && (
              <p className="origin">産地: {recipe.origin}</p>
            )}
            
            <div className="recipe-card-meta">
              <span className="created-at">
                {new Date(recipe.created_at).toLocaleDateString('ja-JP')}
              </span>
            </div>
          </div>
          
          <div className="recipe-card-actions">
            <Link to={`/recipes/${recipe.id}`} className="view-button">
              詳細を見る
            </Link>
          </div>
        </div>
      ))}
    </div>
  );
}
```

## 4. レシピページの更新

`src/pages/recipes/RecipesPage.tsx`を更新：

```tsx
import { useEffect, useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import RecipeList from '../../components/recipe/RecipeList';
import './RecipesPage.css';

export default function RecipesPage() {
  const { user, loading } = useAuth();
  const navigate = useNavigate();
  const [isLoading, setIsLoading] = useState(true);

  // 認証状態を確認し、未ログインならリダイレクト
  useEffect(() => {
    if (!loading && !user) {
      navigate('/login');
    } else if (!loading) {
      setIsLoading(false);
    }
  }, [user, loading, navigate]);

  if (isLoading) {
    return <div className="loading">読み込み中...</div>;
  }

  return (
    <div className="recipes-container">
      <div className="recipes-header">
        <h1>マイレシピ</h1>
        <Link to="/recipes/new" className="create-button">
          新規レシピ作成
        </Link>
      </div>
      
      <RecipeList />
    </div>
  );
}
```

`src/pages/recipes/NewRecipePage.tsx`を作成：

```tsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import RecipeForm from '../../components/recipe/RecipeForm';

export default function NewRecipePage() {
  const { user, loading } = useAuth();
  const navigate = useNavigate();

  // 認証状態を確認し、未ログインならリダイレクト
  useEffect(() => {
    if (!loading && !user) {
      navigate('/login');
    }
  }, [user, loading, navigate]);

  if (loading) {
    return <div className="loading">読み込み中...</div>;
  }

  return (
    <div className="new-recipe-page">
      <RecipeForm />
    </div>
  );
}
```

`src/App.tsx`を更新して新しいルートを追加：

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import MainLayout from './components/layout/MainLayout';
import HomePage from './pages/HomePage';
import LoginPage from './pages/auth/LoginPage';
import SignupPage from './pages/auth/SignupPage';
import RecipesPage from './pages/recipes/RecipesPage';
import NewRecipePage from './pages/recipes/NewRecipePage';
import { AuthProvider } from './hooks/auth/AuthContext';

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<MainLayout />}>
            <Route index element={<HomePage />} />
            <Route path="login" element={<LoginPage />} />
            <Route path="signup" element={<SignupPage />} />
            <Route path="recipes" element={<RecipesPage />} />
            <Route path="recipes/new" element={<NewRecipePage />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```
