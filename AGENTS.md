# 🤖 Agent Catalog

このリポジトリで使用可能なエージェント（自動化レポート生成ワークフロー）の一覧です。

---

## 📊 レポート生成エージェント

| エージェント            | ワークフロー                                                               | 概要                               | トリガー                         |
| ----------------------- | -------------------------------------------------------------------------- | ---------------------------------- | -------------------------------- |
| 🌐 Environment Reporter | [report-environment.yml](.github/workflows/report-environment.yml)         | Azure リソース状態の収集・レポート | 手動 / 週次 / インフラデプロイ後 |
| 💰 Cost Analyzer        | [report-cost.yml](.github/workflows/report-cost.yml)                       | Azure コスト分析と最適化提案       | 手動 / 月次                      |
| 🔐 Security Auditor     | [report-security.yml](.github/workflows/report-security.yml)               | CodeQL/Trivy/シークレット結果集約  | 手動 / 週次 / スキャン完了後     |
| 🔄 Workflow Health      | [report-workflow-health.yml](.github/workflows/report-workflow-health.yml) | GitHub Actions 成功率分析          | 手動 / 週次                      |
| 🔧 Troubleshooter       | [report-troubleshoot.yml](.github/workflows/report-troubleshoot.yml)       | 失敗時の trouble_docs 自動生成     | ワークフロー失敗時 / 手動        |
| 📝 Deploy Diff          | [report-deploy-diff.yml](.github/workflows/report-deploy-diff.yml)         | Bicep What-If 結果の整形           | 手動                             |

---

## 📁 エージェント設計ファイル

エージェントの設計方針・プロンプトは以下に定義されています：

```
.github/
├── agents/                      # エージェント定義
│   ├── orchestrator.agent.md    # オーケストレータ
│   ├── environment-reporter.agent.md
│   ├── cost-analyzer.agent.md
│   ├── security-auditor.agent.md
│   ├── workflow-health.agent.md
│   ├── troubleshooter.agent.md
│   └── deploy-diff.agent.md
│
├── instructions/                # インストラクション
│   ├── agent-design.instructions.md
│   ├── report-format.instructions.md
│   └── workflow-development.instructions.md
│
└── workflows/                   # 実行ワークフロー
    ├── report-environment.yml
    ├── report-cost.yml
    ├── report-security.yml
    ├── report-workflow-health.yml
    ├── report-troubleshoot.yml
    └── report-deploy-diff.yml
```

---

## 🚀 使い方

### 手動実行

1. GitHub の **Actions** タブを開く
2. 実行したいレポートワークフローを選択
3. **Run workflow** をクリック
4. パラメータを入力して実行

### 自動実行

各エージェントは以下のタイミングで自動実行されます：

- **週次**: 環境レポート、セキュリティ監査、ワークフロー健全性
- **月次**: コスト分析
- **イベント駆動**: デプロイ完了後、スキャン完了後、ワークフロー失敗時

---

## 📤 出力先

| 出力形式     | 保存先                     | 用途                   |
| ------------ | -------------------------- | ---------------------- |
| Markdown     | `reports/{type}/{date}.md` | 人間が読むレポート     |
| GitHub Issue | リポジトリの Issues        | 通知・ディスカッション |
| Job Summary  | GitHub Actions UI          | 即時確認               |

---

## 🛠️ カスタマイズ

新しいエージェントを追加する場合：

1. `.github/agents/` にエージェント定義ファイルを作成
2. `.github/workflows/` に対応するワークフローを作成
3. この `AGENTS.md` を更新

詳細は [agent-design.instructions.md](.github/instructions/agent-design.instructions.md) を参照してください。

---

## 📎 関連ドキュメント

- [copilot-instructions.md](.github/copilot-instructions.md) - Copilot 設定
- [README.md](README.md) - プロジェクト概要
- [READMEs/README_WORKFLOWS.md](READMEs/README_WORKFLOWS.md) - ワークフロー詳細
