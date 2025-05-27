{# AIの提案} のような、フロントエンドから Timezone を取得する仕組みを構築してほしい
※ ただし AI の提案は Laravel 11 に対応していない（例えば、kernel.php は Laravle 11 では廃止されている）

フロントエンドからは X-Timezone というキーで Request header に仕込まれる。(実装済み)
X-Timezone が存在しない場合は UTC で処理する

UserQuestionSetService の、summaryUserQuestionSet で Asia/Tokyo でタイムゾーンが決め打ち担っている箇所があるので、この修正に伴って一緒に修正してください。
その他にも タイムゾーンが Asia/Tokyo 決め打ちになっている箇所があればなおしてください

【重要】テストは不要なので  composer test　は実行しないでください。どうしても Testing の実施が必要ならテスト条件は 1 = 1 としてください。（必ずテストに成功する）

↓ # AIの提案
### 1. 仕組みの全体像

| ステップ            | フロントエンド (Vue 3 SPA)                                                                      | バックエンド (Laravel 11 API)                                                                                             |
| --------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **a. タイムゾーン検出** | `Intl.DateTimeFormat().resolvedOptions().timeZone` で IANA 形式のタイムゾーンを取得 (例: `Asia/Tokyo`) | ―                                                                                                                   |
| **b. 送信**       | ① すべての HTTP リクエストにカスタムヘッダー `X-Timezone` を自動付与<br>②（任意）最初のログイン時だけ送信し、ユーザーテーブルに保存          | `DetectClientTimezone` ミドルウェアでヘッダーを読み取り、**リクエスト単位**のタイムゾーンを決定                                                       |
| **c. サーバー処理**   | ―                                                                                        | 1. DB への保存・更新は **必ず UTC**<br>2. 「呼び出し元の“今日”」など、ロジック上クライアント TZ が必要な部分だけ `$tz = request()->get('tz')` で Carbon を生成し計算 |
| **d. レスポンス**    | UTC のまま返す **または** `->setTimezone($tz)->toIso8601String()` で変換して返す                        | フロントで表示用に変換するか、API が変換して返すかは要件次第                                                                                    |

---

### 2. フロントエンド実装例（Vue 3 + Axios）

```ts
// boot/axios.ts
import axios from 'axios';

axios.interceptors.request.use(config => {
  const tz = Intl.DateTimeFormat().resolvedOptions().timeZone; // Asia/Tokyo など
  config.headers['X-Timezone'] = tz;
  return config;
});
```

* **IANA 形式**を送れば DST（夏時間）も自動で解決できます。
* 数値オフセット（`+0900` など）は DST を考慮できないので非推奨。

---

### 3. Laravel 11 実装例

#### 3-1. ミドルウェア

```php
<?php
// app/Http/Middleware/DetectClientTimezone.php

namespace App\Http\Middleware;

use Closure;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class DetectClientTimezone
{
    public function handle(Request $request, Closure $next): Response
    {
        $tz = $request->header('X-Timezone');

        // バリデーション
        if ($tz && in_array($tz, timezone_identifiers_list())) {
            // 必要ならコンテナに格納
            app()->instance('client.timezone', $tz);
        } else {
            // フォールバック
            app()->instance('client.timezone', 'UTC');
        }

        return $next($request);
    }
}
```

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        // …
        \App\Http\Middleware\DetectClientTimezone::class,
    ],
];
```

* `app('client.timezone')` でどこからでも取得可。
* **注意**: `date_default_timezone_set()` でグローバルに変えるとキューや他リクエストに影響するので *リクエストスコープ* のみにする。

#### 3-2. ヘルパやトレイトで Carbon をラップ

```php
trait UsesClientTimezone
{
    protected function clientNow(): \Carbon\CarbonImmutable
    {
        return now(app('client.timezone')); // Laravel 11 は CarbonImmutable が now() 返す
    }

    protected function toClient(Carbon|\DateTimeInterface|null $date): ?string
    {
        return $date?->setTimezone(app('client.timezone'))->toIso8601String();
    }
}
```

使用例（今日タスクを完了したか判定）:

```php
public function completedToday()
{
    $tz   = app('client.timezone');
    $from = now($tz)->startOfDay()->setTimezone('UTC');  // DB は UTC
    $to   = now($tz)->endOfDay()->setTimezone('UTC');

    return Task::whereBetween('finished_at', [$from, $to])->exists();
}
```

#### 3-3. API リソースで返却時に変換

```php
public function toArray($request): array
{
    return [
        'finished_at' => $this->toClient($this->finished_at),
        // 他フィールド…
    ];
}
```

---

### 4. 代替・拡張アプローチ

| 方法                                | 特色                                                       | 注意点                       |
| --------------------------------- | -------------------------------------------------------- | ------------------------- |
| **カスタムヘッダー (`X-Timezone`)**       | 実装がシンプル。キャッシュ戦略で `Vary: X-Timezone` を付与すれば CDN キャッシュも安全。 | 同じブラウザでも拠点を変えるとヘッダー値が変わる。 |
| **クエリパラメータ (`?tz=Asia%2FTokyo`)** | GET リクエストだけなら設定不要。                                       | POST/PUT にも必要だと一貫性が下がる。   |
| **ユーザープロフィールに保持**                 | 一度設定すればヘッダー不要。                                           | マルチデバイス・旅行時にずれる。          |
| **IP ジオロケーション**                   | クライアント改修なし。                                              | VPN・モバイル回線で誤判定／精度低い。      |

—

### 5. まとめ

1. **DB は常に UTC で統一保存**（Laravel デフォルトの `config('app.timezone') => 'UTC'` を維持）
2. SPA からは IANA 形式のタイムゾーンを **`X-Timezone` ヘッダーで送信**。
3. Laravel 11 側でミドルウェアを追加し、`app('client.timezone')` に格納。
4. ロジック・レスポンス変換は Carbon でリクエスト単位に行う。
5. 必要なら `Vary: X-Timezone` で CDN キャッシュ分割を忘れない。

このパターンで **「保存は UTC、処理・表示は呼び出し元タイムゾーン」** を安全に実現できます。



app/Traits/UsesClientTimezone.php
<?php

namespace App\Traits;

use Carbon\Carbon;

/**
 * リクエストのタイムゾーンを取得するTrait
 * X-Timezone ヘッダーに指定がなければ UTC を返す
 */
trait UsesClientTimezone
{
    /**
     * クライアントタイムゾーンを返す
     */
    protected function clientTimezone(): string
    {
        return app()->get('client.timezone', 'UTC');
    }

    /**
     * クライアントタイムゾーンで現在日時を取得
     */
    protected function clientNow(): Carbon
    {
        return now($this->clientTimezone());
    }
}
