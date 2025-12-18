# ワークフロー開発ガイドライン

このプロジェクトの GitHub Actions ワークフロー開発に関するガイドラインです。

---

## 📋 命名規則

### ワークフローファイル

| カテゴリ       | パターン             | 例                             |
| -------------- | -------------------- | ------------------------------ |
| インフラ       | `{番号}-infra-*.yml` | `1-infra-deploy.yml`           |
| アプリ         | `{番号}-{app}-*.yml` | `2-board-app-build-deploy.yml` |
| レポート       | `report-{type}.yml`  | `report-environment.yml`       |
| セキュリティ   | `security-*.yml`     | `security-scan.yml`            |
| ユーティリティ | `{action}.yml`       | `cleanup-workflows.yml`        |

### run-name

```yaml
run-name: "{emoji} {Name} – triggered by ${{ github.actor }}"
```

---

## 🔧 共通パターン

### 環境変数

```yaml
env:
  RESOURCE_GROUP: ${{ vars.RESOURCE_GROUP }}
  ACR_NAME: ${{ vars.ACR_NAME }}
  AKS_NAME: ${{ vars.AKS_NAME }}
```

### Azure 認証

```yaml
- name: Azure Login
  uses: azure/login@v2
  with:
    client-id: ${{ vars.AZURE_CLIENT_ID }}
    tenant-id: ${{ vars.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### エラーハンドリング

```yaml
- name: Step with error handling
  id: step_id
  continue-on-error: true
  run: |
    # 処理

- name: Check result
  if: steps.step_id.outcome == 'failure'
  run: |
    echo "::warning::前のステップが失敗しました"
```

---

## 📊 Job Summary

必ず Job Summary を出力する：

```yaml
- name: Output Summary
  run: |
    echo "## 📊 実行結果" >> $GITHUB_STEP_SUMMARY
    echo "" >> $GITHUB_STEP_SUMMARY
    echo "| 項目 | 値 |" >> $GITHUB_STEP_SUMMARY
    echo "|------|-----|" >> $GITHUB_STEP_SUMMARY
    echo "| 状態 | ✅ 成功 |" >> $GITHUB_STEP_SUMMARY
```

---

## 🔄 トリガー設計

### 手動 + スケジュール

```yaml
on:
  workflow_dispatch:
    inputs:
      resource_group:
        description: "リソースグループ名"
        required: false
        default: "RG-bbs-app-demo"
  schedule:
    - cron: "0 0 * * 1" # 毎週月曜 00:00 UTC
```

### 他ワークフロー完了後

```yaml
on:
  workflow_run:
    workflows: ["1️⃣ Infrastructure Deploy"]
    types:
      - completed
```

---

## 🛡️ セキュリティ

### Permissions

最小権限を指定：

```yaml
permissions:
  contents: read
  issues: write
  actions: read
```

### シークレット

- `AZURE_SUBSCRIPTION_ID`: Secrets
- `AZURE_CLIENT_SECRET`: Secrets
- その他は Variables を使用

---

## 📁 成果物

### アップロード

```yaml
- name: Upload Report
  uses: actions/upload-artifact@v4
  with:
    name: report-${{ github.run_id }}
    path: reports/
    retention-days: 30
```

### reports/ 構造

```
reports/
├── environment/
│   └── 2025-12-18.md
├── cost/
│   └── 2025-12-18.md
├── security/
│   └── 2025-12-18.md
└── workflow-health/
    └── 2025-12-18.md
```
