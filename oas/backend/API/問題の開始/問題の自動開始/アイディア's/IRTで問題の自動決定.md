## アイディアの全体像

| レイヤ | 役割 | 主なクラス/テーブル | 備考 |
|--------|------|--------------------|------|
| **ドメイン** | 学習状態のモデル化 | `UserSkillProficiency` (新規) | ユーザー×スキルの習熟度を確率 p∈[0,1] で保持 |
| **アプリケーション** | 次の問題を決定する | `AdaptiveQuestionSelectorService`<br>`RecommendNextQuestionSetUseCase` | IRT/BKT/Elo のいずれかでアルゴリズムを差し替え可能な Strategy パターン |
| **インフラ** | パラメータ推定・再学習 | Laravel Queue + `RecalibrateDifficultyJob` | 定期バッチで問題パラメータ (難易度・識別力) を更新 |
| **API** | フロントエンドへの提供 | `GET /v1/question-sets/next` | 「次に解くべき問題セット」を JSON で返却 |

---

## 1. 学習状態をどう持つか

### 1-1. スキル階層の明示
- **skills** テーブルを既にお持ちなので、  
  *「かけ算」「分数の足し算」* など最小単位のスキルに Questions をタグ付け (`question_skill` PIVOT)
- スキルは DAG（依存グラフ）で管理すると後述の **Knowledge Space Theory** を組み込みやすいです。

### 1-2. ユーザー習熟度テーブル（追加案）

```mermaid
erDiagram
    users ||--o{ user_skill_proficiencies : has
    skills ||--o{ user_skill_proficiencies : rates

    user_skill_proficiencies {
        id PK
        user_id FK
        skill_id FK
        proficiency DECIMAL(5,4) "0.0000〜1.0000"
        last_updated TIMESTAMP
    }
```

- 初期値は 0.50（未知）で良い。
- 回答毎に **Bayesian Knowledge Tracing** で更新 → `AdaptiveQuestionSelectorService` が書き込み。

---

## 2. 出題アルゴリズムの選択肢

| 難易度 | アルゴリズム | 特長 | 実装コスト |
|------|-------------|------|-----------|
| ★☆☆ | **ヒューリスティック** (正答率ベース) | 直近 N 問のスキル正答率<70% → 類題を出題 | 低 |
| ★★☆ | **Elo/Glicko-2** | Question⇔User をレーティング対戦とみなし能力を数値化 | 中 |
| ★★★ | **IRT (2PL/3PL)** | 難易度(b)・識別力(a)・当て推量(c) の統計モデルで「情報量」を最大化する問題を選択 | 高 |
| ★★★ | **BKT + Knowledge Space** | スキル間依存を考慮し「まだ習得していない prerequisite を優先」 | 高 |

> **最初はヒューリスティックでリリースし、ログを溜めつつ IRT へ移行** が現実的です。

---

## 3. 具体的フロー例（IRT 2PL を採用した場合）

```text
解答送信
  └─ AnswerQuestionUseCase
        ├─ 採点 / スコア算出
        ├─ AdaptiveQuestionSelectorService::updateProficiency() ← IRT θ 更新
        └─ 次の問題取得 API へリダイレクト
              └─ AdaptiveQuestionSelectorService::nextQuestionSet()
                    1. 未完了 question_sets を候補取得
                    2. 各 questions の「情報量 I(θ)」を計算
                    3. 最大 I の question_set を返却
```

### 3-1. θ（能力値）と問題パラメータの更新
- 問題テーブルに `irt_a`(識別力)・`irt_b`(難易度) を追加（NULL はデフォルト a=1.0,b=0.0 扱い）。
- 回答時に **Bayesian 更新**（オンライン推定）
  ```php
  $theta_new = $theta_old + (lr * (is_correct - P_correct($theta_old,a,b)));
  ```
- 夜間バッチ (`RecalibrateDifficultyJob`) で **EMアルゴリズム** を 1 日分まとめて再推定すると精度↑。

### 3-2. 次の問題セット選択
1. `user_question_sets` で **NOT_START** がある → まずはそのセットから順番に出す（学習の一貫性）。
2. 全て完了していれば:
    - ユーザーの苦手スキル上位 K 件を抽出（θ が低い）
    - そのスキルを多く含む `question_sets` を候補に。
    - 候補内で **|b − θ| が最小** の問題を含むセットを優先  
      （成功確率 70% 前後＝最も学習効果が高い帯域）。

---

## 4. 失敗／スキップ時の扱い

| 事象 | DB更新 | 次の出題方針 |
|------|--------|-------------|
| **不正解** | `user_questions.status = WRONG`<br>`θ ↓` | 同スキルだが少し易しい (b ↓) 問題を再出題 |
| **スキップ** | `user_questions.status = SKIP` | 同スキルの別タイプ問題を提案／原因ヒアリング |
| **連続正解 (n≥3)** | `θ ↑↑` | 別スキル or 難度アップ |

---

## 5. テーブル・モデル追加まとめ

| 区分 | 追加/変更 | 目的 |
|------|-----------|------|
| **Migration** | `user_skill_proficiencies` | ユーザー習熟度 |
| | `questions` に `irt_a`,`irt_b`,`irt_c` | IRT パラメータ |
| **Model** | `UserSkillProficiency` | 上記テーブルの Eloquent |
| **Service** | `AdaptiveQuestionSelectorService` | 出題決定ロジック |
| **UseCase** | `RecommendNextQuestionSetUseCase` | API 用ファサード |
| **Job** | `RecalibrateDifficultyJob` | パラメータ再推定 |
| **Enum** | `UserQuestionStatus::WRONG` など | 状態追加が必要なら |

---

## 6. テスト戦略

1. **ユニットテスト**
    - `AdaptiveQuestionSelectorServiceTest`
        - θ=0 のとき b=0 の問題が選ばれるか
        - θ=1 のとき b≃1 の問題が選ばれるか
2. **モデルテスト**
    - `UserSkillProficiency` の `proficiency` が 0〜1 範囲外にならない
3. **E2E** (Feature)
    - あるユーザーが 5 問連続正解 → 難易度が上がることを確認
4. **回帰テスト用シードデータ**
    - `tests/Fixtures/irt_params.csv` を読み込み固定値で検証

---

## 7. スモールスタート手順

1. **ヒューリスティック実装 (2 週間)**
    - 正答率 <60% のスキルを優先するルールで `AdaptiveQuestionSelectorService` 作成
    - ログテーブルに「選択根拠」「所要時間」を保存
2. **データ蓄積 (1–2 か月)**
    - 回答ログ & 所要時間を BigQuery／Redshift へ集約
3. **IRT/BKT PoC (並行)**
    - Python + PyMC でオフライン推定 → a,b パラメータを CSV で import
4. **本番への昇格**
    - 1 回/日の再推定ジョブを Laravel Queue で走らせる
    - フロント UI に「おすすめ度★」を表示し AB テスト

---

## 8. さらなる発展アイディア

- **メタ認知チェック**: 正答後に「自信度」を 1-5 で入力させ、過信/過小診断を校正
- **遅延復習 (Spaced Repetition)**: 忘却曲線に合わせ一定日数後に再出題
- **LLM 生成問題**: `question_sets.generation_blocked = false` かつ **情報量が高いが既存問題不足** のスキルに対し自動生成
- **Explain-why 提示**: 不正解時に「あなたが間違えたのは ___ の概念が弱いから。復習動画はこちら」

---

## 次のステップ

1. **どのアルゴリズムから始めるか決定**
2. 必要なテーブル追加を Migration に落とす
3. `AdaptiveQuestionSelectorService` のインターフェース設計
4. ユニットテストの雛形作成

実装フェーズに進む準備ができたら、追加で **Migration と Service skeleton** を書き起こしますのでお知らせください。
