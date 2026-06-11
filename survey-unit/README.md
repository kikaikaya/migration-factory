# survey-unit — レガシー現状調査の分析ユニット

[flowsmith](https://github.com/KIKAIKAYA/flowsmith) で駆動する、レガシーシステム現状調査の
ワークフロー定義。調査の全工程（資産棚卸し → 分類 → 依存抽出 → 影響判断 → 報告書）を
1 つの YAML で宣言し、エージェントが順に実行する。

## 設計思想

1. **成果物はデータ**。調査結果はすべて列定義済みの CSV
   （[contracts/data-dictionary.md](contracts/data-dictionary.md)）。
   報告書はデータから派生する最終ビューにすぎない。再分析・再利用・Excel 化が自由。
2. **証拠主義**。全行に evidence（根拠の所在）必須。事実（fact）と推定（inferred）を
   列で区別し、確認できないことは確認事項（needs_confirm）として残す。
   **推定を事実と書いた行は検収ゲートで差し戻される。**
3. **順序の強制**。棚卸し → 分類 → 依存抽出 → 影響判断。
   根拠データなしに影響範囲を語る工程は存在しない。

## 実行方法

```bash
# 前提: flowsmith プラグインがインストール済み
# 1. flow.yaml の paths.SRC を調査対象リポジトリに変更
# 2. 実行
claude
> /flowsmith-run survey-unit/flow.yaml
# 中断しても同じコマンドで続きから再開する
```

## 工程一覧

| # | ステップ | 種別 | 出力 |
|---|----------|------|------|
| 0 | 資産の有無確認 | make | 00-asset-check.csv |
| 1 | 資産棚卸しと規模統計 | make | 01-inventory.csv / 01-stats.csv |
| 2 | 実行ドメイン×レイヤー分類 | make-check | 02-layer-map.csv |
| 3 | 外部ライブラリ調査 | make-check | 03-libraries.csv |
| 4 | 依存関係の抽出 | make-check | 04-dependencies.csv / 04-unresolved.csv |
| 5 | 画面側の調査 | make | 05-frontend.csv |
| 6 | 業務処理側の調査 | make | 06-backend.csv |
| 7 | フレームワーク・共通基盤 | make | 07-framework.csv |
| 8 | 言語・実行環境の影響 | make | 08-runtime-impact.csv |
| 9 | DB・SQL の調査 | make-check | 09-db.csv / 09-crud.csv |
| 10 | OS・ビルド・環境依存 | make | 10-platform.csv |
| 11 | 処理フローの追跡 | make-check | 11-flows.csv / flows/*.md |
| 12 | 影響マトリクス | make-check | 12-impact-matrix.csv |
| 13 | データ整合の検収 | branch | （不合格なら 12 へ差し戻し） |
| 14 | 二層報告書 | make-check | report/ + 13-confirm-items.csv |

## 出力構造

```text
survey-out/
├── data/        # 調査データ（CSV。スキーマは contracts/ 参照）
├── flows/       # 処理フロー詳細（Mermaid 図付き）
├── report/      # 二層報告書（意思決定者向け / 技術者向け）
└── .flowsmith/  # 実行状態・ログ（中断再開用）
```
