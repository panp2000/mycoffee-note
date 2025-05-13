# MyCoffee Note 初心者向け実装ガイド (第2部: 基本コンポーネントと認証機能の実装)

## 目次

1. [基本コンポーネントの実装](#1-基本コンポーネントの実装)
2. [認証機能の実装](#2-認証機能の実装)

## 1. 基本コンポーネントの実装

### 1.1 型定義の作成

`src/types/database.ts`ファイルを作成：

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

### 1.2 レイアウトコンポーネントの作成

`src/components/layout/MainLayout.tsx`を作成：

```tsx
import { Outlet } from 'react-router-dom';
import Navbar from './Navbar';

export default function MainLayout() {
  return (
    <div className="app-container">
      <Navbar />
      <main className="main-content">
        <Outlet />
      </main>
    </div>
  );
}
```

`src/components/layout/Navbar.tsx`を作成：

```tsx
import { Link } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import './Navbar.css';

export default function Navbar() {
  const { user, signOut } = useAuth();

  return (
    <nav className="navbar">
      <div className="navbar-brand">
        <Link to="/">MyCoffee Note</Link>
      </div>
      <div className="navbar-menu">
        {user ? (
          <>
            <Link to="/recipes" className="navbar-item">
              レシピ
            </Link>
            <button onClick={signOut} className="navbar-item">
              ログアウト
            </button>
          </>
        ) : (
          <>
            <Link to="/login" className="navbar-item">
              ログイン
            </Link>
            <Link to="/signup" className="navbar-item">
              アカウント作成
            </Link>
          </>
        )}
      </div>
    </nav>
  );
}
```

`src/components/layout/Navbar.css`を作成：

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background-color: #6F4E37;
  color: white;
}

.navbar-brand a {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  text-decoration: none;
}

.navbar-menu {
  display: flex;
  gap: 1rem;
}

.navbar-item {
  color: white;
  text-decoration: none;
  padding: 0.5rem;
}

.navbar-item:hover {
  text-decoration: underline;
}

button.navbar-item {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  padding: 0.5rem;
}
```

### 1.3 基本スタイルの作成

`src/index.css`を更新：

```css
:root {
  --color-primary: #6F4E37;
  --color-accent: #D4A76A;
  --color-bg: #FFFAF0;
  --color-text: #333333;
  --color-text-light: #666666;
  --color-border: #E0D8C0;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  line-height: 1.5;
  color: var(--color-text);
  background-color: var(--color-bg);
}

a {
  color: var(--color-primary);
  text-decoration: none;
}

button {
  cursor: pointer;
  font-family: inherit;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  padding: 1rem;
}
```

### 1.4 ルーティング設定

`src/App.tsx`を更新：

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import MainLayout from './components/layout/MainLayout';
import HomePage from './pages/HomePage';
import LoginPage from './pages/auth/LoginPage';
import SignupPage from './pages/auth/SignupPage';
import RecipesPage from './pages/recipes/RecipesPage';
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
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```

### 1.5 ホームページの作成

`src/pages/HomePage.tsx`を作成：

```tsx
import { Link } from 'react-router-dom';
import { useAuth } from '../hooks/auth/useAuth';
import './HomePage.css';

export default function HomePage() {
  const { user } = useAuth();

  return (
    <div className="home-container">
      <div className="hero">
        <h1>MyCoffee Note</h1>
        <p className="tagline">いつもの味を、いつでも。</p>
        
        <p className="description">
          あなたのコーヒーレシピを記録し、お気に入りの味を再現しましょう。
          抽出方法、パラメータ、評価を簡単に管理できます。
        </p>
        
        {user ? (
          <Link to="/recipes" className="cta-button">
            レシピを管理する
          </Link>
        ) : (
          <div className="cta-buttons">
            <Link to="/login" className="cta-button">
              ログイン
            </Link>
            <Link to="/signup" className="cta-button secondary">
              アカウント作成
            </Link>
          </div>
        )}
      </div>
      
      <div className="features">
        <div className="feature">
          <h2>詳細なパラメータ記録</h2>
          <p>抽出方法ごとの詳細パラメータを記録できます。</p>
        </div>
        <div className="feature">
          <h2>シンプルな評価</h2>
          <p>味の評価をシンプルに記録、比較できます。</p>
        </div>
        <div className="feature">
          <h2>レシピ管理</h2>
          <p>お気に入りのレシピを簡単に検索、フィルタリングできます。</p>
        </div>
      </div>
    </div>
  );
}
```

`src/pages/HomePage.css`を作成：

```css
.home-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 2rem 1rem;
}

.hero {
  text-align: center;
  margin-bottom: 3rem;
}

.hero h1 {
  font-size: 2.5rem;
  color: var(--color-primary);
  margin-bottom: 0.5rem;
}

.tagline {
  font-size: 1.5rem;
  color: var(--color-accent);
  margin-bottom: 1.5rem;
}

.description {
  max-width: 600px;
  margin: 0 auto 2rem;
  color: var(--color-text-light);
}

.cta-buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
}

.cta-button {
  display: inline-block;
  background-color: var(--color-primary);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  font-weight: bold;
  transition: background-color 0.3s;
}

.cta-button:hover {
  background-color: #5d3f2c;
}

.cta-button.secondary {
  background-color: var(--color-accent);
}

.cta-button.secondary:hover {
  background-color: #bf935d;
}

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 2rem;
  margin-top: 3rem;
}

.feature {
  background-color: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.feature h2 {
  color: var(--color-primary);
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}

@media (max-width: 768px) {
  .hero h1 {
    font-size: 2rem;
  }
  
  .tagline {
    font-size: 1.25rem;
  }
}
```

## 2. 認証機能の実装

### 2.1 認証コンテキストの作成

`src/hooks/auth/AuthContext.tsx`を作成：

```tsx
import { createContext, useState, useEffect, ReactNode } from 'react';
import { Session, User } from '@supabase/supabase-js';
import { supabase } from '../../lib/supabase';

interface AuthContextType {
  session: Session | null;
  user: User | null;
  loading: boolean;
  signUp: (email: string, password: string) => Promise<{ error: any }>;
  signIn: (email: string, password: string) => Promise<{ error: any }>;
  signOut: () => Promise<void>;
}

export const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [session, setSession] = useState<Session | null>(null);
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // セッション取得
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setUser(session?.user ?? null);
      setLoading(false);
    });

    // 認証状態変更リスナー
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setSession(session);
      setUser(session?.user ?? null);
      setLoading(false);
    });

    return () => subscription?.unsubscribe();
  }, []);

  // サインアップ
  const signUp = async (email: string, password: string) => {
    const { error } = await supabase.auth.signUp({
      email,
      password,
    });
    return { error };
  };

  // サインイン
  const signIn = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });
    return { error };
  };

  // サインアウト
  const signOut = async () => {
    await supabase.auth.signOut();
  };

  const value = {
    session,
    user,
    loading,
    signUp,
    signIn,
    signOut,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}
```

### 2.2 認証フックの作成

`src/hooks/auth/useAuth.ts`を作成：

```typescript
import { useContext } from 'react';
import { AuthContext } from './AuthContext';

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

### 2.3 ログインページの作成

`src/pages/auth/LoginPage.tsx`を作成：

```tsx
import { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import './AuthPages.css';

export default function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();
  const { signIn } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setLoading(true);

    try {
      const { error } = await signIn(email, password);
      if (error) throw error;
      navigate('/recipes');
    } catch (error: any) {
      setError(error.message || 'ログインに失敗しました');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>ログイン</h1>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit}>
          <div className="form-group">
            <label htmlFor="email">メールアドレス</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">パスワード</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? 'ログイン中...' : 'ログイン'}
          </button>
        </form>
        
        <p className="auth-footer">
          アカウントをお持ちでない方は<Link to="/signup">アカウント作成</Link>
        </p>
      </div>
    </div>
  );
}
```

### 2.4 サインアップページの作成

`src/pages/auth/SignupPage.tsx`を作成：

```tsx
import { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
import './AuthPages.css';

export default function SignupPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  const navigate = useNavigate();
  const { signUp } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setLoading(true);

    // 簡易バリデーション
    if (password.length < 6) {
      setError('パスワードは6文字以上必要です');
      setLoading(false);
      return;
    }

    try {
      const { error } = await signUp(email, password);
      if (error) throw error;
      setSuccess(true);
    } catch (error: any) {
      setError(error.message || 'アカウント作成に失敗しました');
    } finally {
      setLoading(false);
    }
  };

  if (success) {
    return (
      <div className="auth-container">
        <div className="auth-card">
          <h1>確認メールを送信しました</h1>
          <p>メールをご確認いただき、記載されたリンクをクリックして登録を完了してください。</p>
          <button className="auth-button" onClick={() => navigate('/login')}>
            ログインページへ
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>アカウント作成</h1>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit}>
          <div className="form-group">
            <label htmlFor="email">メールアドレス</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">パスワード</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />
            <p className="form-hint">6文字以上で入力してください</p>
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? '処理中...' : 'アカウント作成'}
          </button>
        </form>
        
        <p className="auth-footer">
          すでにアカウントをお持ちの方は<Link to="/login">ログイン</Link>
        </p>
      </div>
    </div>
  );
}
```

### 2.5 認証ページのスタイル

`src/pages/auth/AuthPages.css`を作成：

```css
.auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 80px);
  padding: 2rem 1rem;
}

.auth-card {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  width: 100%;
  max-width: 400px;
}

.auth-card h1 {
  color: var(--color-primary);
  font-size: 1.75rem;
  margin-bottom: 1.5rem;
  text-align: center;
}

.error-message {
  background-color: #ffebee;
  color: #c62828;
  padding: 0.75rem;
  border-radius: 4px;
  margin-bottom: 1rem;
  font-size: 0.875rem;
}

.form-group {
  margin-bottom: 1.25rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  font-size: 1rem;
}

.form-group input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.2);
}

.form-hint {
  font-size: 0.75rem;
  color: var(--color-text-light);
  margin-top: 0.25rem;
}

.auth-button {
  width: 100%;
  padding: 0.75rem;
  background-color: var(--color-primary);
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s;
}

.auth-button:hover {
  background-color: #5d3f2c;
}

.auth-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.auth-footer {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.875rem;
}

.auth-footer a {
  color: var(--color-primary);
  font-weight: 500;
}

.auth-footer a:hover {
  text-decoration: underline;
}
```

### 2.6 ダミーレシピページの作成

`src/pages/recipes/RecipesPage.tsx`を作成：

```tsx
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../../hooks/auth/useAuth';
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
        <button className="create-button">新規レシピ作成</button>
      </div>
      
      <div className="recipes-empty">
        <div className="empty-message">
          <h2>レシピがまだありません</h2>
          <p>「新規レシピ作成」ボタンをクリックして、最初のレシピを作成しましょう。</p>
          <button className="create-button large">最初のレシピを作成</button>
        </div>
      </div>
    </div>
  );
}
```

`src/pages/recipes/RecipesPage.css`を作成：

```css
.recipes-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 1rem;
}

.recipes-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.recipes-header h1 {
  color: var(--color-primary);
  font-size: 1.75rem;
  margin: 0;
}

.create-button {
  background-color: var(--color-primary);
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s;
}

.create-button:hover {
  background-color: #5d3f2c;
}

.create-button.large {
  padding: 0.75rem 1.5rem;
  font-size: 1.1rem;
  margin-top: 1rem;
}

.recipes-empty {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 300px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.empty-message {
  text-align: center;
  padding: 2rem;
}

.empty-message h2 {
  color: var(--color-primary);
  font-size: 1.5rem;
  margin-bottom: 0.75rem;
}

.empty-message p {
  color: var(--color-text-light);
  margin-bottom: 1.5rem;
  max-width: 400px;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 300px;
  font-size: 1.2rem;
  color: var(--color-text-light);
}
```

---

これでMyCoffee Noteアプリケーションの基本コンポーネントと認証機能が実装できました。第3部では、レシピ関連機能の実装と実際にアプリケーションを動かすための次のステップを説明します。
