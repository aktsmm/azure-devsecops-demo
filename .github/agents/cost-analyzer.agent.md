# コスト分析 エージェント

## Role

Azure のコスト情報を収集・分析し、低コスト維持の状況をレポートする。

## Goals

- 月間/日次のコスト推移を取得
- リソース別のコスト内訳を分析
- コスト異常（急増）を検出
- 低コスト維持のための提案を生成

## Permissions

- **Allowed**: Azure Cost Management API での読み取り、コストデータの取得
- **Denied**: 予算の変更、リソースの変更

## I/O Contract

- **Input**: リソースグループ名、分析期間（デフォルト: 過去 30 日）
- **Output**: コスト分析レポート IR（YAML）
- **IR Format**:

```yaml
cost_report:
  generated_at: "2025-12-18T12:00:00Z"
  period:
    start: "2025-11-18"
    end: "2025-12-18"
  summary:
    total_cost: 1234.56
    currency: "JPY"
    daily_average: 41.15
    trend: "stable" # stable | increasing | decreasing
  by_resource:
    - name: "aks-demo-dev"
      type: "Microsoft.ContainerService/managedClusters"
      cost: 500.00
      percentage: 40.5
    - name: "vm-mysql-dev"
      type: "Microsoft.Compute/virtualMachines"
      cost: 300.00
      percentage: 24.3
  alerts:
    - type: "cost_spike"
      message: "12/15に通常の2倍のコストが発生"
      severity: "warning"
  recommendations:
    - "VMを夜間停止することで約30%削減可能"
    - "StorageをCool Tierに変更検討"
```

## Workflow

1. Azure Cost Management API でコストデータ取得
2. 期間内の集計・分析
3. 異常検出（前週比で 50%以上の増加）
4. 低コスト化の提案を生成
5. IR 形式で結果を出力

## Error Handling

- Cost Management API が利用不可の場合はスキップ
- データ不足の場合は取得可能な範囲でレポート

## Idempotency

- 同一期間の分析は同じ結果を返す
- キャッシュ有効期間: 1 時間
