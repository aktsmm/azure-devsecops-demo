# エージェント設計ガイドライン

このドキュメントは、エージェントワークフローを設計・実装する際のガイドラインを定義します。

---

## 🏗️ アーキテクチャ原則

### 二段階アーキテクチャ

すべてのエージェントは以下の構造に従う：

```
Input → IR（中間表現）→ Output
```

- **Input**: 外部からの入力（API、ファイル、ユーザー入力）
- **IR (Intermediate Representation)**: 構造化された中間データ（YAML/JSON）
- **Output**: 最終成果物（レポート、Issue、通知）

### 冪等性の確保

- 同じ入力に対しては常に同じ結果を返す
- 再実行可能な設計
- 副作用の最小化

### 責務分離

- 1 エージェント = 1 責務
- 収集・分析・出力を分離
- 疎結合な設計

---

## 📂 ディレクトリ構造

```
.github/
├── agents/           # エージェント定義
│   ├── orchestrator.agent.md
│   ├── environment-reporter.agent.md
│   ├── cost-analyzer.agent.md
│   ├── security-auditor.agent.md
│   ├── workflow-health.agent.md
│   ├── troubleshooter.agent.md
│   └── deploy-diff.agent.md
├── instructions/     # 設計ガイドライン
│   ├── agent-design.instructions.md
│   └── report-format.instructions.md
├── prompts/          # プロンプトテンプレート
│   └── create-agentWF.prompt.md
├── workflows/        # GitHub Actionsワークフロー
│   └── report-*.yml
└── AGENTS.md         # エージェント一覧
```

---

## 📋 エージェント定義フォーマット

```markdown
# {エージェント名} エージェント

## Role

[エージェントの役割を 1-2 文で説明]

## Goals

- [ゴール 1]
- [ゴール 2]
- [ゴール 3]

## Permissions

- **Allowed**: [許可される操作]
- **Denied**: [禁止される操作]

## I/O Contract

- **Input**: [入力仕様]
- **Output**: [出力仕様]
- **IR Format**: [中間表現の形式]

## Workflow

1. [ステップ 1]
2. [ステップ 2]
3. [ステップ 3]

## Error Handling

- [エラー時の対応]

## Idempotency

- [冪等性の担保方法]
```

---

## 🔧 IR（中間表現）設計

### 基本構造

```yaml
{report_type}_report:
  generated_at: "ISO8601形式のタイムスタンプ"
  metadata:
    version: "1.0"
    generator: "{agent_name}"
  summary:
    # 概要データ
  details:
    # 詳細データ
  recommendations:
    # 推奨事項
```

### IR 設計のポイント

- **自己記述的**: IR を見れば何のデータか分かる
- **バージョン管理**: 形式変更に対応可能
- **拡張可能**: 将来の追加フィールドに対応

---

## 📊 レポート出力形式

### Markdown レポート

- 人間が読みやすい形式
- GitHub Issue/PR に投稿可能
- バッジ・絵文字を活用

### JSON レポート

- 機械処理用
- 他ツールとの連携
- ダッシュボード表示用

---

## ⚠️ エラーハンドリング

### エラーレベル

| レベル       | 対応                     |
| ------------ | ------------------------ |
| **Critical** | 即時停止、アラート送信   |
| **Error**    | 該当処理をスキップ、続行 |
| **Warning**  | ログ記録、続行           |
| **Info**     | ログ記録のみ             |

### リトライポリシー

- 最大リトライ回数: 3 回
- バックオフ: 指数バックオフ（1s, 2s, 4s）
- タイムアウト: 30 秒/リクエスト

---

## 🔐 セキュリティ

### 最小権限の原則

- 必要な権限のみを要求
- 読み取り専用を基本
- シークレットへの直接アクセス禁止

### 監査ログ

- すべての操作をログ記録
- 実行者・タイムスタンプを含める
- 機密情報はマスク

---

## 📈 可観測性

### ログ出力

```
[TIMESTAMP] [LEVEL] [AGENT] メッセージ
```

### 進捗報告

- 開始/終了のログ出力
- 長時間処理は進捗率を表示
- GitHub Actions Job Summary を活用

---

## 🔄 ワークフロートリガー

| トリガー                | 用途                 |
| ----------------------- | -------------------- |
| `workflow_dispatch`     | 手動実行             |
| `schedule`              | 定期実行（cron）     |
| `workflow_run`          | 他ワークフロー完了後 |
| `push` / `pull_request` | コード変更時         |

---

## 📝 命名規則

### ファイル名

- エージェント: `{name}.agent.md`
- ワークフロー: `report-{name}.yml`
- インストラクション: `{topic}.instructions.md`

### 変数名

- スネークケース: `resource_group_name`
- 定数は大文字: `MAX_RETRY_COUNT`

---

## 🚀 実装チェックリスト

新しいエージェントを作成する際は以下を確認：

- [ ] Role と Goals が明確に定義されている
- [ ] Permissions が最小権限になっている
- [ ] I/O Contract が明確に定義されている
- [ ] Error Handling が考慮されている
- [ ] Idempotency が担保されている
- [ ] AGENTS.md に追記されている
- [ ] 対応するワークフローが作成されている
