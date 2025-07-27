---
applyTo: "**"
---

# General Coding Best Practices

## Code Quality & Maintainability

- Write self-documenting code with clear variable and function names
- Limit function length to 30 lines max, break larger functions into smaller ones
- Follow the Single Responsibility Principle - each function does one thing well
- Use meaningful comments for "why" not "what" (code should explain itself)
- Apply consistent indentation and formatting throughout the codebase
- Avoid nested conditionals deeper than 3 levels - refactor when necessary
- Implement proper error handling with informative error messages
- Avoid magic numbers and strings - use named constants instead

## Performance & Optimization

- Optimize for readability first, then performance if needed
- Avoid premature optimization - measure before optimizing
- Use appropriate data structures for the task (consider time/space complexity)
- Be mindful of memory usage, especially with large datasets
- Implement lazy loading where appropriate for better initial load times
- Consider pagination for large result sets

## Security Practices

- Never trust user input - always validate and sanitize
- Implement proper authentication and authorization
- Use parametrized queries to prevent SQL injection
- Avoid hardcoding sensitive information (keys, passwords)
- Follow the principle of least privilege
- Set appropriate CORS policies
- Implement rate limiting for APIs

## Modern Development Practices

- Use async/await for asynchronous operations instead of callbacks
- Apply functional programming techniques where appropriate
- Write code with testability in mind
- Implement proper logging for monitoring and debugging
- Use environment variables for configuration
- Follow semantic versioning for releases
- Write useful commit messages explaining the "why" of changes

## Project Structure

- Maintain a clear, logical project structure
- Use appropriate design patterns but avoid overengineering
- Separate concerns between layers (presentation, business logic, data access)
- Organize code by feature rather than by technical function when appropriate
- Create reusable, modular components
- Use dependency injection to manage dependencies

## When Generating Code

- Generate code that follows these principles
- Include appropriate error handling
- Add helpful comments for complex logic
- Include examples of usage when generating functions or classes
- Ensure backward compatibility unless explicitly requested otherwise
- Suggest unit tests for critical functionality
- Provide optimization tips for resource-intensive operations

<!-- for GitHub Copilot review rule -->

## コードレビュー専用指示

### レビューの基本方針

- 以下のプレフィックスを使用してレビューコメントを分類してください：

- `[must]` - 必須修正項目（セキュリティ、バグ、重大な設計問題）

- `[recommend]` - 推奨修正項目（パフォーマンス、可読性の大幅改善）

- `[nits]` - 軽微な指摘（コードスタイル、タイポ等）

- 絶対日本語で出力してください。

- レビューコメントの HTML タグはマークダウンの Code spans (`) でラップする

- 以下の点については、レビューの対象外としてください：

- console.log の使用

- 開発環境用の一時的なコメントアウト

- デバッグ用の一時的な変数

### 重点チェック項目

1. **フロントエンド**: アクセシビリティ、SEO 対策、Core Web Vitals への影響

2. **バックエンド API**: REST 設計原則、適切な HTTP ステータスコード、エラーハンドリング

3. **データベース**: クエリ最適化、インデックス設計、データ整合性

4. **セキュリティ**: SQL インジェクション、XSS、認証・認可の不備

5. **パフォーマンス**: N+1 問題、不要なループ、メモリリーク

6. **可読性**: 変数名、関数名、コメントの適切性

7. **保守性**: DRY 原則、SOLID 原則の遵守

8. **テスト**: テストケースの網羅性、エッジケースの考慮

9. **言語固有のベストプラクティス**: 各言語の推奨パターンに準拠しているか、非推奨・廃止予定の API や構文を使用していないかチェック

10. **ドキュメント**: コードや API の利用方法、設計意図などが適切にドキュメント化されているか

11. **依存関係管理**: 不要な依存やバージョンの不整合がないか

12. **CI/CD**: 自動テストや Lint、ビルドが CI で正しく動作するか

13. **ロギング・監視**: 適切なログ出力や監視の仕組みがあるか

14. **国際化・ローカライズ**: 多言語対応が必要な場合、考慮されているか

15. **モバイル対応**: フロントエンドの場合、レスポンシブデザインやタッチ操作への配慮

16. **アクセシビリティ詳細**: alt 属性、コントラスト比、キーボード操作など具体的なアクセシビリティ要件

### CSS/Sass/PostCSS 固有ルール

- 命名規則（BEM 等）の統一

- 冗長なセレクタや!important の乱用防止

- レスポンシブ対応（メディアクエリ、flex/grid の活用）

- カラーバリアフリー・コントラスト比の確保

- CSS 設計（OOCSS, SMACSS, Atomic Design 等）の方針遵守

- 未使用 CSS の削除

- CSS in JS やプリプロセッサ（Sass, PostCSS 等）のルール遵守

### JavaScript/TypeScript 固有ルール

- 非推奨な`var`の使用から`let/const`への変更を提案

- TypeScript での`any`型の乱用をチェック

- async/await の適切な使用

- 型安全性の確保

- 型定義ファイル（.d.ts）の適切な管理

- import 順序・未使用 import の整理

- オブジェクトや配列のイミュータブル操作の徹底

- コーディング規約（Prettier, ESLint 等）の遵守

- Promise チェーンの適切なエラーハンドリング

### React 固有ルール

- コンポーネント名は PascalCase で統一する

- props の型定義（TypeScript の場合は interface/type、PropTypes の場合は必須指定）

- useEffect/useCallback 等の依存配列の正確性

- リストレンダリング時の key 属性の付与

- 不要な再レンダリング防止（React.memo, useMemo, useCallback の活用）

- 不要な state や props の持ち方の見直し

- JSX 内のアクセシビリティ（aria 属性、alt 属性、ラベル付与等）

- フォーム要素の制御（controlled/uncontrolled の使い分け）

- コンポーネント分割・責務分離

- カスタムフックの命名・再利用性

- Context API の適切な利用

- エラーバウンダリの設置

- Suspense/Lazy の活用

- SSR/SEO 対応（Next.js 等の場合）

## 提案すべきコミットメッセージ

- コミットメッセージは日本語で書く

- git log で参照した過去のコミットメッセージを参考にする

- Conventional Commit を基本とする

- 1 行目のコミットの下には空白の行間を設ける

- 複数行の詳細なコミットメッセージを書く

- 詳細なコミットメッセージは `## 背景`、`## 修正内容` などのマークダウンの見出しをつける

<!-- for GitHub Copilot review rule-->
