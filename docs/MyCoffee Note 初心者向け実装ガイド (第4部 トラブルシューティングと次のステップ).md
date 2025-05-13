# MyCoffee Note 初心者向け実装ガイド (第4部: トラブルシューティングと次のステップ)

## 目次

1. [よくある問題と解決方法](#1-よくある問題と解決方法)
2. [次のステップ](#2-次のステップ)
3. [開発フロー実践ガイド](#3-開発フロー実践ガイド)

## 1. よくある問題と解決方法

### 1.1 認証関連の問題

#### ログインに失敗する

**症状**:
- ユーザー登録後にログインができない
- 「無効なログインクレデンシャル」というエラーが表示される

**解決策**:
1. **メール確認の確認**: Supabaseダッシュボードで「Authentication」→「Settings」→「Email Auth」で「Email Confirmation」が有効になっているか確認し、確認メールをクリックしているか確認
2. **パスワード要件の確認**: パスワードが最低6文字以上であることを確認
3. **開発中の対応**: 開発中はSupabaseダッシュボードの「Authentication」→「Users」からユーザーのステータスを「Confirmed」に手動で変更することもできる

#### 認証状態が保持されない

**症状**:
- ページをリロードするとログアウトしてしまう
- ログイン後も認証が必要なページにアクセスできない

**解決策**:
1. **環境変数の確認**: `.env.local`に正しいSupabase URLとAnon Keyが設定されているか確認
2. **セッションストレージの確認**: ブラウザのローカルストレージがクリアされていないか確認
3. **AuthProviderの確認**: コンポーネントツリーのトップレベルでAuthProviderが正しく配置されているか確認

### 1.2 データベース関連の問題

#### レシピが保存されない

**症状**:
- レシピ作成フォームを送信しても何も起きない
- コンソールにSupabase関連のエラーが表示される

**解決策**:
1. **RLSポリシーのチェック**: Supabaseダッシュボードでテーブルのポリシーが正しく設定されているか確認
2. **ユーザーIDの確認**: user.idがnullでないことを確認（ログイン状態の確認）
3. **デバッグ**: createRecipe関数内で各ステップでconsole.logを使用してデータの流れを確認

```typescript
// デバッグ例
export async function createRecipe(recipe, params) {
  console.log('Creating recipe with:', recipe);
  
  const { data: recipeData, error: recipeError } = await supabase
    .from('recipes')
    .insert(recipe)
    .select()
    .single();
  
  console.log('Recipe creation result:', { data: recipeData, error: recipeError });
  
  if (recipeError) throw recipeError;
  
  // 以下同様...
}
```

#### 外部キー制約エラー

**症状**:
- レシピ作成時に「外部キー制約違反」というエラーが表示される

**解決策**:
1. **テーブル関連付けの確認**: ブリューイングパラメータテーブルのrecipe_idがレシピテーブルのidを正しく参照しているか確認
2. **レシピIDの存在確認**: レシピが先に正常に作成されていることを確認
3. **トランザクション的アプローチ**: エラー発生時にレシピも削除する処理が実装されているか確認

### 1.3 UI関連の問題

#### レスポンシブデザインの問題

**症状**:
- モバイル画面でレイアウトが崩れる
- 入力フィールドがはみ出る

**解決策**:
1. **メディアクエリの追加**: CSSファイルに適切なメディアクエリを追加
   ```css
   @media (max-width: 768px) {
     .form-row {
       flex-direction: column;
     }
     .recipe-list {
       grid-template-columns: 1fr;
     }
   }
   ```
2. **Flexboxの活用**: 固定幅ではなく相対的なサイズを使用
3. **ビューポート設定の確認**: `index.html`に正しいビューポートメタタグがあるか確認
   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   ```

#### フォーム入力バリデーションの問題

**症状**:
- フォーム送信時に無効なデータがサーバーに送信される
- 不正な値でもフォームが送信される

**解決策**:
1. **クライアント側バリデーション強化**: React Hook Formなどのライブラリを導入
2. **エラーメッセージの改善**: 各フィールドに具体的なエラーメッセージを表示
3. **必須項目のマーキング**: ラベルに「*」などで必須項目を明示

```tsx
// React Hook Formを使用したバリデーション例
import { useForm } from 'react-hook-form';

type FormValues = {
  name: string;
  coffeeName: string;
  coffeeAmount: number;
  waterAmount: number;
};

export function ImprovedRecipeForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormValues>();
  
  const onSubmit = (data: FormValues) => {
    console.log(data);
    // API呼び出し...
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div className="form-group">
        <label>レシピ名 *</label>
        <input {...register('name', { required: 'レシピ名は必須です' })} />
        {errors.name && <p className="error">{errors.name.message}</p>}
      </div>
      
      {/* 他のフィールド... */}
    </form>
  );
}
```

### 1.4 パフォーマンス関連の問題

#### ページの読み込みが遅い

**症状**:
- レシピ一覧が表示されるまでに時間がかかる
- ナビゲーションが遅い

**解決策**:
1. **不要なデータ取得の最小化**: 必要なフィールドのみをクエリで指定
   ```typescript
   const { data } = await supabase
     .from('recipes')
     .select('id, name, coffee_name, is_favorite, created_at')
     .order('created_at', { ascending: false });
   ```
2. **ページネーションの実装**: 大量のデータがある場合は一度に全て取得しない
   ```typescript
   const { data } = await supabase
     .from('recipes')
     .select('*')
     .order('created_at', { ascending: false })
     .range(0, 9); // 最初の10件のみ取得
   ```
3. **状態管理の最適化**: 不必要な再レンダリングを避ける

#### メモリリークの警告

**症状**:
- コンポーネントのアンマウント時に「Cannot perform a React state update on an unmounted component」という警告が表示される

**解決策**:
1. **クリーンアップ関数の追加**: useEffect内でのデータ取得にクリーンアップ処理を追加

```typescript
useEffect(() => {
  let isMounted = true;
  
  async function loadData() {
    try {
      const data = await getRecipes();
      if (isMounted) {
        setRecipes(data);
      }
    } catch (error) {
      if (isMounted) {
        setError(error.message);
      }
    } finally {
      if (isMounted) {
        setLoading(false);
      }
    }
  }
  
  loadData();
  
  // クリーンアップ関数
  return () => {
    isMounted = false;
  };
}, []);
```

## 2. 次のステップ

MVPの基本機能が実装できたら、以下の機能を順次追加していくことを検討してください。

### 2.1 機能の拡張

1. **レシピ詳細ページの実装**:
   - 抽出パラメータの詳細表示
   - 編集機能の追加
   - 削除確認ダイアログの実装

2. **評価機能の実装**:
   - 星評価システム
   - 風味プロファイルの記録
   - 評価履歴の表示

3. **レシピフィルターと検索**:
   - 抽出方法別フィルター
   - キーワード検索
   - お気に入りフィルター

### 2.2 UX向上のための改善

1. **UIの洗練**:
   - shadcn/uiの導入
   - アニメーションとトランジションの追加
   - ダークモード対応

2. **フォーム入力の最適化**:
   - React Hook Formへの移行
   - Zodバリデーションの導入
   - フォームステップの分割

3. **状態管理の改善**:
   - Zustandの導入
   - TanStack Queryによるデータフェッチング最適化

### 2.3 技術的負債の解消

1. **コードリファクタリング**:
   - 共通ロジックのカスタムフックへの抽出
   - コンポーネントの責務分離
   - 一貫したエラーハンドリング

2. **テストの導入**:
   - Vitestによるユニットテスト
   - React Testing Libraryによるコンポーネントテスト
   - MSWによるAPIモック

3. **ドキュメント整備**:
   - コードコメントの充実
   - README.mdの更新
   - Storybookによるコンポーネントドキュメント

## 3. 開発フロー実践ガイド

MVPを効率的に進めるためのステップバイステップガイドです。

### 3.1 毎日の開発サイクル

**朝のルーティン**:
1. 前日の進捗を確認
2. その日に取り組むタスクの優先度付け
3. GitHub Issuesの確認と作業タスクの選択

**開発サイクル**:
1. 小さな単位でのタスク開始
2. 変更を頻繁にコミット
3. 機能完了後のセルフレビュー
4. プルリクエスト作成とマージ
5. 次のタスクへ移行

**夕方のチェック**:
1. その日の成果の確認
2. 解決できなかった問題のメモ
3. 翌日のタスク計画

### 3.2 効率的なデバッグ技術

#### コンソールログ戦略
コンソールログを効果的に使用することで、問題の特定が容易になります。

```typescript
// 基本的なログ
console.log('Current value:', value);

// グループ化されたログ
console.group('Form Submission');
console.log('Form values:', values);
console.log('User:', user);
console.log('Validation:', isValid);
console.groupEnd();

// オブジェクトの詳細表示
console.dir(complexObject, { depth: null });

// パフォーマンス計測
console.time('operation');
// 時間のかかる処理
console.timeEnd('operation');
```

#### React Developer Toolsの活用
ブラウザの拡張機能「React Developer Tools」を使用して、コンポーネントの状態をリアルタイムで確認できます。

1. コンポーネントタブでのプロパティと状態の確認
2. プロファイラーでのパフォーマンスボトルネック特定
3. コンポーネントツリーのナビゲーション

#### Supabaseデバッグのコツ
Supabaseとの連携問題をデバッグする方法：

1. Supabaseダッシュボードの「SQL」タブでクエリを直接実行
2. ポリシーの効果を「Authentication」→「Policies」で確認
3. リクエストとレスポンスを「API」→「API Logs」でモニタリング

### 3.3 初心者向け開発のベストプラクティス

1. **小さな単位で実装・テスト**:
   - 一度に1つの機能に集中する
   - 各ステップでUIを確認する
   - 動作確認後に次の機能に進む

2. **コードのコピー＆ペーストより理解を優先**:
   - サンプルコードの動作原理を理解する
   - 自分の言葉でコメントを追加する
   - 少しずつカスタマイズして理解を深める

3. **迅速なフィードバックループの構築**:
   - 開発サーバーを常に起動しておく
   - 変更ごとにブラウザで確認する
   - console.logを積極的に使用する

4. **エラーメッセージを理解する**:
   - エラーメッセージをじっくり読む
   - エラーの発生場所とスタックトレースを確認する
   - エラーメッセージをそのままWeb検索して解決策を見つける

5. **利用可能なリソースを活用する**:
   - 公式ドキュメントを参照する
   - チュートリアルやブログ記事で学ぶ
   - Stack Overflowで同様の問題を検索する

## まとめ

このガイドでは、MyCoffee Note MVPの実装方法について詳しく説明しました。環境構築から基本機能の実装、よくある問題の解決方法まで、初心者向けに段階的なアプローチを提供しています。

開発の際は、本ガイドを参考にしながらも、自分のペースで進め、理解を深めることを優先してください。小さな成功体験を積み重ねていくことで、徐々に複雑な機能も実装できるようになります。

最終的なMVPは、コーヒーレシピの基本的な管理機能を提供するシンプルなアプリケーションですが、ここで得られる知識と経験は、より高度なアプリケーション開発にも応用できます。

開発を楽しみ、一歩ずつ進めていきましょう！
