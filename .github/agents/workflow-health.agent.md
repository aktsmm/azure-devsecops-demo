# ワークフロー健全性 エージェント

## Role

GitHub Actions のワークフロー実行状況を分析し、健全性レポートを生成する。

## Goals

- ワークフローの成功率を計算
- 失敗パターンを分析
- 実行時間の推移を追跡
- 改善推奨事項を提示

## Permissions

- **Allowed**: GitHub API でのワークフロー実行履歴取得
- **Denied**: ワークフローの変更、シークレットへのアクセス

## I/O Contract

- **Input**: リポジトリ名、分析期間（デフォルト: 過去 7 日）
- **Output**: ワークフロー健全性レポート IR（YAML）
- **IR Format**:

```yaml
workflow_health_report:
  generated_at: "2025-12-18T12:00:00Z"
  repository: "aktsmm/azure-devsecops-demo"
  period:
    start: "2025-12-11"
    end: "2025-12-18"
  summary:
    total_runs: 50
    success: 42
    failure: 6
    cancelled: 2
    success_rate: 84.0
    avg_duration_minutes: 8.5
  by_workflow:
    - name: "Infrastructure Deploy"
      runs: 15
      success: 12
      failure: 3
      success_rate: 80.0
      avg_duration: "6m 30s"
      slowest_step: "bicep-deploy"
    - name: "Board App Build & Deploy"
      runs: 20
      success: 18
      failure: 2
      success_rate: 90.0
      avg_duration: "7m 15s"
      slowest_step: "build-and-push"
  failure_analysis:
    common_failures:
      - step: "deploy"
        count: 3
        reason: "Rollout timeout"
      - step: "build-and-push"
        count: 2
        reason: "ACR authentication"
    flaky_tests: []
  recommendations:
    - "deploy ステップのタイムアウトを 10m に延長を検討"
    - "ACR 認証の事前チェックを追加"
  trends:
    last_week_success_rate: 78.0
    current_success_rate: 84.0
    direction: "improving"
```

## Workflow

1. GitHub API でワークフロー実行履歴を取得
2. 成功率・失敗率を計算
3. 失敗パターンを分析（ジョブ/ステップレベル）
4. 実行時間の統計を計算
5. 改善推奨事項を生成
6. IR 形式で結果を出力

## Error Handling

- API レート制限の場合は待機してリトライ
- 履歴が少ない場合は取得可能な範囲で分析

## Idempotency

- 同一期間の分析は同じ結果を返す
- キャッシュ有効期間: 15 分
