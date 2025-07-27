# お問い合わせフォームシステム - 設計書

## 1. システム設計概要

### 1.1 システム構成図

```
[Next.js Frontend]  ←→  [Laravel Backend]  ←→  [MySQL Database]
    (Vercel)              (Railway)           (Railway MySQL)
```

### 1.2 アーキテクチャ

-   **フロントエンド**: Next.js (SSR/CSR) + Tailwind CSS
-   **バックエンド**: Laravel 10 + Sanctum API
-   **データベース**: MySQL 8.0
-   **認証**: Laravel Breeze + Sanctum Token
-   **通信**: RESTful API (JSON)

## 2. データベース設計

### 2.1 ER 図

```
Users (管理者)                Contact Forms (お問い合わせ)
├─ id (PK)                   ├─ id (PK)
├─ name                      ├─ name
├─ email                     ├─ email
├─ password                  ├─ subject
├─ created_at                ├─ message
└─ updated_at                ├─ status (enum: 'pending', 'exported')
                             ├─ exported_at
                             ├─ created_at
                             └─ updated_at
```

### 2.2 テーブル定義

#### 2.2.1 users テーブル

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified_at TIMESTAMP NULL,
    password VARCHAR(255) NOT NULL,
    remember_token VARCHAR(100) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    INDEX idx_email (email)
);
```

#### 2.2.2 contact_forms テーブル

```sql
CREATE TABLE contact_forms (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    subject VARCHAR(500) NOT NULL,
    message TEXT NOT NULL,
    status ENUM('pending', 'exported') DEFAULT 'pending',
    exported_at TIMESTAMP NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
);
```

## 3. API 設計

### 3.1 エンドポイント一覧

#### 3.1.1 認証 API

```
POST /api/login          # ログイン
POST /api/logout         # ログアウト
GET  /api/user           # 認証ユーザー情報取得
```

#### 3.1.2 お問い合わせ API

```
POST /api/contact-forms                    # お問い合わせ送信（公開）
GET  /api/admin/contact-forms             # お問い合わせ一覧取得（認証必要）
POST /api/admin/contact-forms/export      # CSVエクスポート（認証必要）
DELETE /api/admin/contact-forms/{id}      # お問い合わせ削除（認証必要）
DELETE /api/admin/contact-forms/bulk      # 一括削除（認証必要）
```

### 3.2 API 仕様詳細

#### 3.2.1 お問い合わせ送信 API

```http
POST /api/contact-forms
Content-Type: application/json

{
  "name": "山田太郎",
  "email": "yamada@example.com",
  "subject": "お問い合わせの件",
  "message": "詳細なメッセージ内容..."
}

Response (201):
{
  "message": "お問い合わせを受け付けました。",
  "data": {
    "id": 1,
    "name": "山田太郎",
    "email": "yamada@example.com",
    "subject": "お問い合わせの件",
    "message": "詳細なメッセージ内容...",
    "status": "pending",
    "created_at": "2025-07-27T10:30:00Z"
  }
}
```

#### 3.2.2 お問い合わせ一覧取得 API

```http
GET /api/admin/contact-forms?status=pending&page=1&per_page=20
Authorization: Bearer {token}

Response (200):
{
  "data": [
    {
      "id": 1,
      "name": "山田太郎",
      "email": "yamada@example.com",
      "subject": "お問い合わせの件",
      "message": "詳細なメッセージ内容...",
      "status": "pending",
      "created_at": "2025-07-27T10:30:00Z"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 45,
    "last_page": 3
  }
}
```

#### 3.2.3 CSV エクスポート API

```http
POST /api/admin/contact-forms/export
Authorization: Bearer {token}
Content-Type: application/json

{
  "ids": [1, 2, 3] // 省略時は未処理全件
}

Response (200):
{
  "message": "エクスポートが完了しました。",
  "filename": "contact_forms_2025-07-27_10-30-15.csv",
  "download_url": "/download/contact_forms_2025-07-27_10-30-15.csv",
  "exported_count": 3
}
```

## 4. フロントエンド設計

### 4.1 ディレクトリ構成

```
src/
  app/
    page.tsx                  # トップページ（お問い合わせフォーム）
    (app)/                    # 管理画面
      dashboard/
        page.tsx              # 管理画面トップ
    (auth)/                   # 認証関連
      login/
        page.tsx              # ログインページ
      register/
        page.tsx              # 新規登録ページ
      ...                     # その他認証関連ページ
    LoginLinks.tsx
    global.css
    layout.tsx
    not-found.tsx
  components/
    ApplicationLogo.tsx
    Button.tsx
    Dropdown.tsx
    DropdownLink.tsx
    Input.tsx
    InputError.tsx
    Label.tsx
    NavLink.tsx
    ResponsiveNavLink.tsx
  hooks/
    auth.ts
  lib/
    axios.ts
```

### 4.2 状態管理設計

> 補足: 状態管理用のカスタムフックは `src/hooks/` ディレクトリに配置します。

例: `src/hooks/useAuth.ts`, `src/hooks/useContactForms.ts`

#### 4.2.1 認証状態

```javascript
// useAuth hook
const useAuth = () => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    const login = async (credentials) => {
        /* ... */
    };
    const logout = async () => {
        /* ... */
    };

    return { user, loading, login, logout };
};
```

#### 4.2.2 お問い合わせ管理状態

```javascript
// useContactForms hook
const useContactForms = () => {
    const [forms, setForms] = useState([]);
    const [currentTab, setCurrentTab] = useState("pending");
    const [selectedIds, setSelectedIds] = useState([]);
    const [loading, setLoading] = useState(false);

    const fetchForms = async (status, page) => {
        /* ... */
    };
    const exportForms = async (ids) => {
        /* ... */
    };
    const deleteForms = async (ids) => {
        /* ... */
    };

    return {
        forms,
        currentTab,
        selectedIds,
        loading,
        fetchForms,
        exportForms,
        deleteForms,
    };
};
```

## 5. バックエンド設計

### 5.1 ディレクトリ構成

```
app/
├─ Http/
│  ├─ Controllers/
│  │  ├─ Auth/
│  │  │  └─ AuthController.php
│  │  ├─ ContactFormController.php
│  │  └─ Admin/
│  │     └─ ContactFormController.php
│  ├─ Requests/
│  │  ├─ ContactFormRequest.php
│  │  └─ Admin/
│  │     └─ ExportRequest.php
│  └─ Resources/
│     ├─ ContactFormResource.php
│     └─ ContactFormCollection.php
├─ Models/
│  ├─ User.php
│  └─ ContactForm.php
├─ Services/
│  └─ ContactFormService.php
└─ Exports/
   └─ ContactFormsExport.php
```

### 5.2 主要クラス設計

#### 5.2.1 ContactForm モデル

```php
<?php

namespace App\Models;

class ContactForm extends Model
{
    protected $fillable = [
        'name', 'email', 'subject', 'message', 'status', 'exported_at'
    ];

    protected $casts = [
        'exported_at' => 'datetime',
    ];

    public function scopePending($query)
    {
        return $query->where('status', 'pending');
    }

    public function scopeExported($query)
    {
        return $query->where('status', 'exported');
    }

    public function markAsExported()
    {
        $this->update([
            'status' => 'exported',
            'exported_at' => now(),
        ]);
    }
}
```

#### 5.2.2 ContactFormService クラス

```php
<?php

namespace App\Services;

class ContactFormService
{
    public function createContactForm(array $data): ContactForm
    {
        return ContactForm::create($data);
    }

    public function getPaginatedForms(string $status, int $perPage = 20)
    {
        return ContactForm::where('status', $status)
            ->orderBy('created_at', 'desc')
            ->paginate($perPage);
    }

    public function exportToCSV(array $ids = null): string
    {
        $query = ContactForm::pending();

        if ($ids) {
            $query->whereIn('id', $ids);
        }

        $forms = $query->get();

        // CSVエクスポート処理
        $filename = 'contact_forms_' . now()->format('Y-m-d_H-i-s') . '.csv';

        // ステータス更新
        $forms->each->markAsExported();

        return $filename;
    }

    public function deleteForms(array $ids): int
    {
        return ContactForm::exported()->whereIn('id', $ids)->delete();
    }
}
```

### 5.3 バリデーション設計

#### 5.3.1 ContactFormRequest

```php
<?php

namespace App\Http\Requests;

class ContactFormRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|max:255',
            'subject' => 'required|string|max:500',
            'message' => 'required|string|max:5000',
        ];
    }

    public function messages()
    {
        return [
            'name.required' => 'お名前は必須です。',
            'email.required' => 'メールアドレスは必須です。',
            'email.email' => 'メールアドレスの形式が正しくありません。',
            'subject.required' => '件名は必須です。',
            'message.required' => 'メッセージは必須です。',
        ];
    }
}
```

## 6. セキュリティ設計

### 6.1 認証・認可

#### 6.1.1 Sanctum 設定

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    env('APP_URL') ? ','.parse_url(env('APP_URL'), PHP_URL_HOST) : ''
))),

'expiration' => 525600, // 1年
```

#### 6.1.2 CORS 設定

```php
// config/cors.php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:3000')],
'allowed_origins_patterns' => [],
'allowed_headers' => ['*'],
'exposed_headers' => [],
'max_age' => 0,
'supports_credentials' => true,
```

### 6.2 レート制限設定

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

protected $routeMiddleware = [
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];

// routes/api.php
Route::post('/contact-forms', [ContactFormController::class, 'store'])
    ->middleware('throttle:5,1'); // 1分間に5回まで
```

### 6.3 入力値サニタイゼーション

```php
// ContactFormService.php
public function sanitizeInput(array $data): array
{
    return [
        'name' => strip_tags(trim($data['name'])),
        'email' => filter_var(trim($data['email']), FILTER_SANITIZE_EMAIL),
        'subject' => strip_tags(trim($data['subject'])),
        'message' => strip_tags(trim($data['message'])),
    ];
}
```

## 7. パフォーマンス設計

### 7.1 データベース最適化

#### 7.1.1 インデックス設計

```sql
-- contact_forms テーブルのインデックス
CREATE INDEX idx_contact_forms_status ON contact_forms(status);
CREATE INDEX idx_contact_forms_created_at ON contact_forms(created_at);
CREATE INDEX idx_contact_forms_status_created_at ON contact_forms(status, created_at);
```

#### 7.1.2 クエリ最適化

```php
// 効率的なページネーション
ContactForm::select(['id', 'name', 'email', 'subject', 'created_at'])
    ->where('status', $status)
    ->orderBy('created_at', 'desc')
    ->paginate(20);

// メッセージの省略表示用
ContactForm::selectRaw('*, SUBSTRING(message, 1, 100) as message_preview')
    ->where('status', $status)
    ->orderBy('created_at', 'desc')
    ->paginate(20);
```

### 7.2 フロントエンド最適化

#### 7.2.1 Next.js 最適化設定

```javascript
// next.config.js
module.exports = {
    experimental: {
        optimizeCss: true,
    },
    images: {
        optimization: true,
    },
    compress: true,
};
```

#### 7.2.2 コンポーネント最適化

```javascript
// React.memo を使用した最適化
const ContactFormItem = React.memo(({ form, onSelect }) => {
    return (
        <tr>
            <td>{form.name}</td>
            <td>{form.email}</td>
            <td>{form.subject}</td>
            <td>{form.created_at}</td>
        </tr>
    );
});

// useMemo を使用したデータ加工の最適化
const processedForms = useMemo(() => {
    return forms.map((form) => ({
        ...form,
        messagePreview: form.message.substring(0, 100),
        formattedDate: new Date(form.created_at).toLocaleString("ja-JP"),
    }));
}, [forms]);
```

## 8. エラーハンドリング設計

### 8.1 バックエンドエラーハンドリング

#### 8.1.1 例外ハンドラー

```php
// app/Exceptions/Handler.php
public function render($request, Throwable $exception)
{
    if ($request->expectsJson()) {
        if ($exception instanceof ValidationException) {
            return response()->json([
                'message' => 'バリデーションエラーが発生しました。',
                'errors' => $exception->validator->errors(),
            ], 422);
        }

        if ($exception instanceof ModelNotFoundException) {
            return response()->json([
                'message' => 'リソースが見つかりません。',
            ], 404);
        }

        return response()->json([
            'message' => 'サーバーエラーが発生しました。',
        ], 500);
    }

    return parent::render($request, $exception);
}
```

### 8.2 フロントエンドエラーハンドリング

#### 8.2.1 API エラーハンドリング

```javascript
// hooks/useApi.js
const useApi = () => {
    const handleApiError = (error) => {
        if (error.response?.status === 422) {
            // バリデーションエラー
            return error.response.data.errors;
        } else if (error.response?.status === 401) {
            // 認証エラー
            router.push("/login");
        } else {
            // その他のエラー
            toast.error(
                "エラーが発生しました。しばらく経ってから再度お試しください。"
            );
        }
    };

    return { handleApiError };
};
```

## 9. テスト設計

### 9.1 バックエンドテスト

#### 9.1.1 単体テスト

```php
// tests/Unit/ContactFormServiceTest.php
class ContactFormServiceTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_create_contact_form()
    {
        $service = new ContactFormService();
        $data = [
            'name' => 'テスト太郎',
            'email' => 'test@example.com',
            'subject' => 'テスト件名',
            'message' => 'テストメッセージ',
        ];

        $form = $service->createContactForm($data);

        $this->assertInstanceOf(ContactForm::class, $form);
        $this->assertEquals('pending', $form->status);
    }
}
```

#### 9.1.2 統合テスト

```php
// tests/Feature/ContactFormApiTest.php
class ContactFormApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_submit_contact_form()
    {
        $data = [
            'name' => 'テスト太郎',
            'email' => 'test@example.com',
            'subject' => 'テスト件名',
            'message' => 'テストメッセージ',
        ];

        $response = $this->postJson('/api/contact-forms', $data);

        $response->assertStatus(201)
                ->assertJsonStructure([
                    'message',
                    'data' => ['id', 'name', 'email', 'subject', 'message']
                ]);
    }

    public function test_requires_authentication_for_admin_endpoints()
    {
        $response = $this->getJson('/api/admin/contact-forms');
        $response->assertStatus(401);
    }
}
```

### 9.2 フロントエンドテスト

#### 9.2.1 コンポーネントテスト

```javascript
// __tests__/ContactForm.test.js
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import ContactForm from "../components/Forms/ContactForm";

describe("ContactForm", () => {
    test("submits form with valid data", async () => {
        const mockSubmit = jest.fn();
        render(<ContactForm onSubmit={mockSubmit} />);

        fireEvent.change(screen.getByLabelText("お名前"), {
            target: { value: "テスト太郎" },
        });
        fireEvent.change(screen.getByLabelText("メールアドレス"), {
            target: { value: "test@example.com" },
        });
        fireEvent.change(screen.getByLabelText("件名"), {
            target: { value: "テスト件名" },
        });
        fireEvent.change(screen.getByLabelText("メッセージ"), {
            target: { value: "テストメッセージ" },
        });

        fireEvent.click(screen.getByText("送信"));

        await waitFor(() => {
            expect(mockSubmit).toHaveBeenCalledWith({
                name: "テスト太郎",
                email: "test@example.com",
                subject: "テスト件名",
                message: "テストメッセージ",
            });
        });
    });
});
```

## 10. デプロイ設計

### 10.1 環境構成

#### 10.1.1 開発環境

-   **フロントエンド**: `npm run dev` (localhost:3000)
-   **バックエンド**: `php artisan serve` (localhost:8000)
-   **データベース**: Docker MySQL 8.0

#### 10.1.2 本番環境

-   **フロントエンド**: Vercel
-   **バックエンド**: Railway
-   **データベース**: Railway MySQL

### 10.2 環境変数設計

#### 10.2.1 Next.js 環境変数

```bash
# .env.local
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
NEXT_PUBLIC_APP_NAME=お問い合わせフォーム
```

#### 10.2.2 Laravel 環境変数

```bash
# .env
APP_NAME="Contact Form API"
APP_ENV=production
APP_KEY=base64:your-key-here
APP_DEBUG=false
APP_URL=https://api.yourdomain.com

DB_CONNECTION=mysql
DB_HOST=your-db-host
DB_PORT=3306
DB_DATABASE=your-database
DB_USERNAME=your-username
DB_PASSWORD=your-password

SANCTUM_STATEFUL_DOMAINS=yourdomain.com
FRONTEND_URL=https://yourdomain.com
SESSION_DOMAIN=.yourdomain.com
```

### 10.3 CI/CD 設計

#### 10.3.1 GitHub Actions (Laravel)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Railway

on:
    push:
        branches: [main]

jobs:
    deploy:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            - name: Setup PHP
              uses: shivammathur/setup-php@v2
              with:
                  php-version: "8.1"
            - name: Install dependencies
              run: composer install --no-dev --optimize-autoloader
            - name: Run tests
              run: php artisan test
            - name: Deploy to Railway
              run: |
                  curl -f -X POST \
                    -H "Authorization: Bearer ${{ secrets.RAILWAY_TOKEN }}" \
                    "https://backboard.railway.app/graphql/v2" \
                    -d '{"query": "mutation { deploymentTrigger(input: { projectId: \"${{ secrets.RAILWAY_PROJECT_ID }}\", environmentId: \"${{ secrets.RAILWAY_ENVIRONMENT_ID }}\" }) { id } }"}'
```

## 11. 監視・ログ設計

### 11.1 ログ設計

#### 11.1.1 アプリケーションログ

```php
// ログレベル別出力
Log::info('Contact form submitted', ['email' => $email, 'ip' => $request->ip()]);
Log::warning('Rate limit exceeded', ['ip' => $request->ip()]);
Log::error('Database connection failed', ['exception' => $e->getMessage()]);
```

#### 11.1.2 ログローテーション

```php
// config/logging.php
'daily' => [
    'driver' => 'daily',
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'days' => 30,
],
```

### 11.2 監視設計

#### 11.2.1 ヘルスチェック

```php
// routes/api.php
Route::get('/health', function () {
    return response()->json([
        'status' => 'ok',
        'timestamp' => now(),
        'services' => [
            'database' => DB::connection()->getPdo() ? 'ok' : 'error',
            'cache' => Cache::has('health-check') ? 'ok' : 'error',
        ]
    ]);
});
```

---

**文書作成日**: 2025 年 7 月 27 日  
**文書バージョン**: 1.0  
**作成者**: システム設計担当者  
**レビュー者**: シニアエンジニア
