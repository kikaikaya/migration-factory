# legacy-migration-skills

レガシーシステムの**現状調査・マイグレーション評価**を Claude Code で自動化する Skill 集。

「2025年の崖」を越えるための最初の一歩——「このシステム、中身どうなってるの？」に答える作業を、AI エージェントに任せます。

## なにができるか

任意のレガシーコードベース（Java 中心）に対して：

| Skill | 役割 |
|-------|------|
| `legacy-survey` | **調査ワークフロー全体のオーケストレーター**。下記 Skill を順に実行し、最終報告書まで出す |
| `tech-inventory` | 技術スタックの棚卸し（言語/フレームワーク/ビルド/バージョン/EOL チェック） |
| `structure-map` | 構造と依存関係の可視化（モジュール構成・エントリポイント・レイヤー分析） |
| `db-migration-assess` | データ層の調査と移行評価（Oracle → PostgreSQL 互換性チェックポイント） |
| `survey-report` | As-Is 調査報告書の生成（日本語・経営層/開発者の両向け） |

## 使い方

```bash
# Claude Code のプラグインとしてインストール
/plugin install <このリポジトリ>

# または skills/ を手動で ~/.claude/skills/ にコピー

# 調査対象のリポジトリで実行
claude
> /legacy-survey
```

個別の Skill だけ使うこともできます（例：DB 移行評価だけ欲しい → `/db-migration-assess`）。

## 設計方針

- **モデル非依存**：Skill は Markdown の手順書。特定モデルにロックインしない
- **手順はすべて実務由来**：レガシーマイグレーション現場での調査手法を一般化・再構成したもの。特定の組織・案件の情報は一切含まない
- **報告書ファースト**：調査の終着点は「人が読んで意思決定できる日本語の報告書」。中間生成物もすべて Markdown で残す

## 方法論

調査の考え方（なぜこの順番か、何を見るか）は [docs/methodology.md](docs/methodology.md) を参照。

## ステータス

🚧 **v0.1 — 骨格構築中。** Skill の手順は随時、実務知見を反映して深化させます。

## License

MIT © kikaikaya
