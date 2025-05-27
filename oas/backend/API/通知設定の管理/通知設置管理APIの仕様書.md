以下に条件を追加してください。

# 追加条件 
1
LEARNING_RESULT, NO_LEARNING_REMINDER のような、notification_type は ENUM で管理する

2
メール、LINE のような　channel　も ENUM で管理する

3
GET　PATCH　いずれのAPIでも呼び出し時に、
user に紐づく user_notification_settings
が存在しない場合は、データを作成する。
データは、notification_type の ENUM に指定されいる値毎に、channel に登録されている値を全て。
例）
NO_LEARNING_REMINDER　の channel （メール、LINE）をデフォルトは



--- 
# 通知管理 API 仕様書

最終更新: 2025-05-27
担当: **backend team**

---

## 1. 用語と構造

| 用語 / エンティティ                      | 説明                                                                                                    |
| -------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **user\_notification\_settings** | 1 ユーザーにつき 1 レコード。各種通知のオン/オフや、通知全体の停止フラグを保持する設定テーブル。                                                   |
| **notification\_type**           | 通知の分類。現時点では `LEARNING_RESULT`（学習結果通知）と `NO_LEARNING_REMINDER`（未学習リマインド）の 2 種を実装。今後の追加を想定し ENUM として定義。 |
| **channel**                      | 送信経路。メールと LINE を想定しているが、通知オプションは「経路」ではなく「通知種別」単位で管理する。経路の有効/無効は各種通知サービス側で制御。                          |

#### テーブル構成（概念図）

```
users (id PK)
   │1 ── 1
user_notification_settings (id PK, user_id FK → users)
   ├─ learning_result_enabled        bool
   ├─ no_learning_reminder_enabled   bool
   ├─ all_notifications_paused       bool
   └─ updated_at                     datetime
```

> **ポイント**
>
> * `all_notifications_paused = true` のときは、個別フラグの値に関わらず全ての通知を送信しない。
> * 追加の通知種別が増えた場合はカラムを追加するか、JSON カラムに移行してもよい。

---

## 2. エンドポイント一覧

| Method    | Path                                            | 説明             |
| --------- | ----------------------------------------------- | -------------- |
| **GET**   | `/api/v1/users/{user_id}/notification-settings` | 指定ユーザーの通知設定を取得 |
| **PATCH** | `/api/v1/users/{user_id}/notification-settings` | 通知設定を更新（部分更新）  |

> 認可: いずれも **本人または管理者** のみアクセス可。

---

## 3. リクエスト & レスポンス

### 3-1. 取得（GET）

#### リクエスト例

```
GET /api/v1/users/123/notification-settings
Authorization: Bearer {token}
```

#### レスポンス 200

```json
{
  "user_id": 123,
  "learning_result_enabled": true,
  "no_learning_reminder_enabled": false,
  "all_notifications_paused": false,
  "updated_at": "2025-05-27T18:05:00+09:00"
}
```

| フィールド                          | 型        | 説明                   |
| ------------------------------ | -------- | -------------------- |
| `learning_result_enabled`      | bool     | 学習結果通知を送る場合 `true`   |
| `no_learning_reminder_enabled` | bool     | 未学習リマインドを送る場合 `true` |
| `all_notifications_paused`     | bool     | `true` のとき全通知停止      |
| `updated_at`                   | datetime | 最終更新日時（ISO-8601）     |

---

### 3-2. 更新（PATCH）

#### リクエスト例

（学習結果通知をオフ、未学習リマインドをオン、まとめ停止はオフのまま）

```
PATCH /api/v1/users/123/notification-settings
Authorization: Bearer {token}
Content-Type: application/json

{
  "learning_result_enabled": false,
  "no_learning_reminder_enabled": true
}
```

> **部分更新**: 送られてきたキーのみを更新。その他の設定は変更しない。

#### レスポンス 200

リクエスト後の最新状態を JSON で返す（フォーマットは 3-1 と同じ）。

#### バリデーション & ステータスコード

| ステータス               | 条件               |
| ------------------- | ---------------- |
| **200 OK**          | 正常更新             |
| **400 Bad Request** | body が空、または不正型   |
| **403 Forbidden**   | 本人/管理者以外がアクセス    |
| **404 Not Found**   | `user_id` が存在しない |

---

## 4. ビジネスルール

1. `all_notifications_paused = true` のときは **全通知をスキップ**

    * キュー投入前に **共通ヘルパ** `NotificationGuard::shouldSend($userId, $notificationType)` で判定。
    * 判定ロジック<br>

      ```
      if (settings.all_notifications_paused) return false;
      if ($notificationType === LEARNING_RESULT) return settings.learning_result_enabled;
      if ($notificationType === NO_LEARNING_REMINDER) return settings.no_learning_reminder_enabled;
      ```
2. 設定変更後は **即時適用**。すでに送信待ちのジョブには影響しない（簡易実装）。

    * 将来的にジョブ側で再チェックすることで厳密化可能。
3. デフォルト値（ユーザー新規登録時）

   | フィールド                          | 初期値     |
      | ------------------------------ | ------- |
   | `learning_result_enabled`      | `true`  |
   | `no_learning_reminder_enabled` | `true`  |
   | `all_notifications_paused`     | `false` |

---

## 5. 実装ガイド

| 項目              | 推奨実装                                                                                            |
| --------------- | ----------------------------------------------------------------------------------------------- |
| **モデル**         | `UserNotificationSetting` (Eloquent)                                                            |
| **FormRequest** | `UpdateNotificationSettingRequest` — bool 型のみ許可                                                 |
| **Service**     | 現状の `notifyUserQuestionSet`, `CheckLearningStatusJob` で `NotificationGuard::shouldSend()` を呼び出す |
| **マイグレーション**    | users テーブルと 1:1 のため、独立テーブル推奨（将来の拡張・履歴管理のため）                                                     |
| **テスト**         | <ul><li>設定反映の API テスト (Feature)</li><li>送信ガードのユニットテスト</li></ul>                                 |

---

## 6. 将来拡張メモ

* **通知経路別の ON/OFF**

    * 例: LINE だけ停止したい要望が増えたら `channel_settings` JSON を追加。
* **通知時間帯の指定**

    * 午後のみ受け取りたい ⇒ `quiet_hours` (from/to) を追加し、ジョブ側でフィルタ。
* **履歴テーブル**

    * `notification_logs` を導入し、送信結果と設定スナップショットを保存すると分析に役立つ。

---

### END
