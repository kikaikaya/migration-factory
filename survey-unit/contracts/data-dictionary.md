# データ契約 — 分析ユニット出力スキーマ

> 分析ユニットの全出力 CSV の列定義。**この契約が上流（分析）と下流（再構築）の接合面**。
> 列の追加は可（後方互換）、列の削除・改名・意味変更は契約違反。
> CSV は UTF-8・ヘッダ行必須・カンマ区切り（値内カンマは `"` で囲む）。

## 全 CSV 共通の規約

| 列 | 値 | 意味 |
|----|----|------|
| `evidence` | パス / `file:line` / 設定項目名 | この行の根拠。**空欄禁止**（根拠が無い事実は書かない） |
| `confidence` | `fact` / `inferred` | fact=資産から直接確認 / inferred=命名・配置等からの推定 |
| `needs_confirm` | `yes` / `no` | 人間・顧客への確認が必要か |
| `note` | 自由記述 | 補足（推定の場合は判断根拠を書く） |

**三値原則**: 事実（fact）・推定（inferred）・確認事項（needs_confirm=yes）を必ず区別する。
推定を事実と書いた行は検収で差し戻される。

## 各ファイルのスキーマ

### 00-asset-check.csv — 資産の有無

| 列 | 説明 |
|----|------|
| `asset_kind` | source / ui-resource / batch-shell / config / library / build / db / deploy-env / document |
| `status` | present / partial / missing |
| `limitation` | missing/partial の場合、制限される後続調査 |

### 01-inventory.csv — 資産棚卸し（1 行 = 1 ファイル）

| 列 | 説明 |
|----|------|
| `path` | リポジトリルートからの相対パス |
| `name` | ファイル名 |
| `ext` | 拡張子 |
| `language` | java / jsp / html / js / css / xml / properties / sql / shell / other |
| `lines` | 実測行数 |

### 01-stats.csv — 規模統計

| 列 | 説明 |
|----|------|
| `language` | 言語 |
| `files` | ファイル数 |
| `total_lines` | 総行数 |

### 02-layer-map.csv — 実行ドメイン×レイヤー分類

| 列 | 説明 |
|----|------|
| `path` | 資産パス（01-inventory.csv と結合キー） |
| `domain` | `online-ui` / `online-logic` / `batch` / `cross`（横断） / `unknown` |
| `layer` | `app` / `framework` / `library` / `language` / `runtime` / `database` / `os` / `infra` |
| `asset_type` | source / config / library / sql / shell / build / deploy / document |

### 03-libraries.csv — 外部ライブラリ

| 列 | 説明 |
|----|------|
| `name` | ライブラリ名 |
| `version` | バージョン（不明なら空欄 + needs_confirm=yes） |
| `vendor` | 提供元 |
| `oss` | yes / no / unknown |
| `purpose` | 機能の一行説明 |
| `used_by` | 利用箇所の代表（モジュール・パッケージ） |
| `risk` | 移行リスク（EOL・後継・互換性）。high / mid / low / none |

### 04-dependencies.csv — 依存関係（1 行 = 1 依存）

| 列 | 説明 |
|----|------|
| `source` | 依存元（クラス.メソッド / ファイル） |
| `source_type` | class / method / jsp / config / shell / sql / job |
| `dep_type` | `call` / `extends` / `implements` / `dispatch` / `reflection` / `annotation` / `config` / `taglib` / `library` / `sql` / `file` / `batch` |
| `target` | 依存先 |
| `target_type` | class / method / table / file / url / library / job |

### 04-unresolved.csv — 未解決依存

| 列 | 説明 |
|----|------|
| `source` | 追跡を試みた起点 |
| `reason` | 追えなかった理由（動的生成・資産欠落・権利制限 等） |

### 05-frontend.csv — 画面資産（1 行 = 1 画面/部品）

| 列 | 説明 |
|----|------|
| `path` | 画面資産パス |
| `kind` | page / fragment / script / style |
| `backend_link` | 接続先（URL / コントローラ）。複数は `;` 区切り |
| `components` | 利用部品（タグライブラリ等） |
| `browser_dependency` | 特定ブラウザ・プラグイン依存の内容（無ければ none） |
| `risk` | high / mid / low / none |

### 06-backend.csv — 業務処理クラス（1 行 = 1 クラス）

| 列 | 説明 |
|----|------|
| `class` | 完全修飾クラス名 |
| `role` | entry（入口） / logic / validation / db-access / data / common / fw-extension |
| `biz_category` | 業務分類（推定可。その場合 confidence=inferred） |
| `framework_dep` | 依存フレームワーク |
| `library_dep` | 依存外部ライブラリ |
| `db_access` | yes / no |
| `common` | yes / no（多数から参照される共通部品か。根拠は被参照数） |

### 07-framework.csv — フレームワーク・共通基盤

| 列 | 説明 |
|----|------|
| `name` | 対象名 |
| `origin` | oss / in-house / unknown |
| `usage_style` | extends / call / config-driven（複数可 `;` 区切り） |
| `usage_count` | 利用箇所数（04-dependencies.csv 由来） |
| `replace_impact` | 置換影響 high / mid / low + 一行説明 |

### 08-runtime-impact.csv — 言語・実行環境影響（1 行 = 1 影響箇所）

| 列 | 説明 |
|----|------|
| `area` | language-version / namespace-migration / deprecated-api / server-api / library-compat |
| `location` | 影響箇所（クラス / import / 設定） |
| `current` | 現行の使い方 |
| `impact` | 何が起きるか（コンパイル不可 / 動作変更 / パッケージ変更 等） |
| `risk` | high / mid / low |

### 09-db.csv — DB オブジェクト・SQL（1 行 = 1 オブジェクト/SQL 群）

| 列 | 説明 |
|----|------|
| `source` | SQL の所在（クラス.メソッド / SQL ファイル） |
| `object_kind` | sql / procedure / function / trigger / view / sequence / link / encoding |
| `object_name` | 対象 DB オブジェクト名 |
| `specific` | DB 製品固有の関数・構文（無ければ none） |
| `risk` | high / mid / low / none |

### 09-crud.csv — CRUD 対応（1 行 = 1 プログラム×テーブル）

| 列 | 説明 |
|----|------|
| `program` | クラス / SQL ファイル |
| `table` | テーブル名 |
| `crud` | C/R/U/D の組合せ（例: `CR`, `R`, `CRUD`） |

### 10-platform.csv — 環境依存点（1 行 = 1 依存点）

| 列 | 説明 |
|----|------|
| `location` | 依存箇所（ファイル / スクリプト / 設定） |
| `kind` | shell / file-path / os-command / env-var / encoding / build / deploy |
| `detail` | 依存の内容（ハードコードパスの値 等） |
| `impact_area` | 環境変更時に壊れる領域 |
| `risk` | high / mid / low |

### 11-flows.csv — 処理フロー一覧（詳細は flows/{flow_id}.md）

| 列 | 説明 |
|----|------|
| `flow_id` | FLOW-001 形式 |
| `biz_name` | 業務名（業務の言葉で） |
| `kind` | online / batch |
| `entry` | 入口（画面 / ジョブ起動） |
| `route_summary` | 経路の要約（入口→…→DB） |
| `touches` | 触る DB テーブル・ファイル |
| `binding` | direct（直接呼出のみ） / config-driven / mixed |

### 12-impact-matrix.csv — 影響マトリクス（1 行 = 1 影響）

| 列 | 説明 |
|----|------|
| `dep_from` | 依存元（影響を受ける側） |
| `dep_to` | 依存先（変化する側: framework / library / language / runtime / database / browser / os / infra） |
| `action` | `migrate` / `replace` / `upgrade` / `none`（理由必須） / `confirm` |
| `impact` | 影響内容 |
| `risk` | high / mid / low |
| `basis` | 根拠データ（CSV 名 + 行の特定情報）。**上流に無い事実の新規記載は禁止** |
| `policy` | 対応方針案 |

### 13-confirm-items.csv — 確認事項一覧（全 CSV の needs_confirm=yes の集約）

| 列 | 説明 |
|----|------|
| `item` | 確認したいこと |
| `origin` | 出所（CSV 名 + 行の特定情報） |
| `reason` | なぜ資産から確認できないか |
| `ask_to` | 確認先の想定（顧客 / 運用 / ベンダー / 技術検証） |
