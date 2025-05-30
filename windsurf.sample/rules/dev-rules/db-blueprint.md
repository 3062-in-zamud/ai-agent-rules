---
description:
globs:
alwaysApply: true
---
---
description: Universal database rules & guidelines — agnostic to any specific RDBMS or ORM.
globs:
  - "**/*"
alwaysApply: true
---

まず、このファイルを参照したら、この **`##### DBルール確認完了！ #####`** を発言すること

# データベース設計ルール v2.1（DB/ORM 共通）

> **目的** — どの RDBMS（PostgreSQL, MySQL, MariaDB, SQLite, SQL Server, Oracle …）や
> **ORM / マイグレーションツール**（Prisma, TypeORM, Sequelize, Hibernate, SQLAlchemy, Flyway, Liquibase …）を採用しても
> 「同じ設計思想」と「同じ開発フロー」で運用できるようにする。

---

## 0. エンジン & 接続ポリシー

| 環境 | 推奨エンジン例 | 方針 |
|------|---------------|------|
| **Local / CI** | SQLite, Dockerized PostgreSQL | “依存レス & スピード優先” |
| **Staging / Prod** | PostgreSQL ≥ 14 / MySQL 8 など | 耐障害性・拡張性優先 |

- **`DATABASE_URL`** など 1 変数で接続先を切替える設計を徹底。
- エンジン固有機能を使う場合は **代替策** を設計書に明記（例: PG の `JSONB` → MySQL は `JSON`）。

---

## 1. モデル / テーブル命名規則

| レイヤ | ルール |
|--------|-------|
| エンティティクラス名 | 単数形パスカルケース: `User` |
| 実テーブル名 | 複数形スネークケース: `users` |
| 中間テーブル | `<left>_<right>` のアルファベット順 |

> **例** — `User` ↔︎ `Role` (M:N) → `user_roles`

---

## 2. カラム命名規則

| 用途 | 命名 | 型の推奨 |
|------|------|---------|
| PK | `id` | UUID v4 (char 36) もしくは BIGINT |
| FK | `<parent>_id` | 同 parent PK と同型 |
| 生成/更新 | `created_at` / `updated_at` | `TIMESTAMP WITH TIME ZONE` |
| 論理削除 | `deleted_at` (NULL=未削除) | 同上 |
| バージョン | `version` | INTEGER (楽観的ロック用) |

---

## 3. データ型ガイドライン

| 目的 | 汎用 SQL 型 | 備考 |
|------|-------------|------|
| 金額 | `NUMERIC(20,2)` |
| 小数 | `DECIMAL` / `DOUBLE PRECISION` |
| 真偽 | `BOOLEAN` |
| JSON | `JSON` / `JSONB` (PG) |
| 列挙 | 独立テーブル or DB‑native ENUM (可搬性に注意) |

> **NOTE** — SQLite など型制約が緩い DB ではアプリ & テストで **型 round‑trip** を必ず検証。

---

## 4. リレーション & 削除戦略

| 関係 | 実装指針 |
|------|---------|
| 1 : N | 子に FK + INDEX。削除は論理削除が原則 |
| N : M | 中間テーブル + `UNIQUE(left_id,right_id)` |
| 1 : 1 | 片側に `UNIQUE` |
| 物理削除 | 外部依存なく、監査不要のときのみ `ON DELETE CASCADE` |

---

## 5. インデックス & パフォーマンス

1. **FK** には自動または手動で INDEX。
2. 頻出 WHERE / ORDER BY 列に単一 or 複合 INDEX。
3. 追加 INDEX は “**90/10 ルール**” — 全クエリの 90 % をカバー。
4. 影響は EXPLAIN / ANALYZE で確認し、**Before/After** を設計書に添付。

---

## 6. バリデーション層

| 階層 | 役割 |
|------|------|
| DB 制約 | NOT NULL, CHECK, UNIQUE, FK |
| ORM レイヤ | 型・範囲・正規表現 |
| サービス層 | 業務ルール (二重予約など) |

> **三重防御** で安全性を担保。

---

## 7. マイグレーション管理

1. **ツール自由**（例: Prisma Migrate, Flyway, Liquibase, Alembic）。
2. ファイル命名: `<timestamp>__<summary>.sql` or `<tool>_<summary>`。
3. マイグレ内で **DDL / DML** を分離し、ロールバックを必ず定義。
4. CI で “**空 DB → 最新 → ロールバック → 再適用**” テストを実施。
5. 本番適用は **トランザクション or Online DDL** を使用し、Downtime 0 を目指す。

---

## 8. セキュリティ & 機密データ

- **機密列** はアプリ側 AES‑GCM 等で暗号化。
- ハッシュは `argon2id` + Pepper（ENV）。
- アプリ/ORM レイヤで **行レベルセキュリティ** (RLS) を実装する場合は DB 側 POLICY と同期。

---

## 9. 可観測性 & N+1 対策

| 項目 | 推奨ツール |
|------|-----------|
| Query Trace | `pg_stat_statements`, MySQL `slow_query_log`, ORM Logger |
| APM | OpenTelemetry + Grafana / Datadog |
| バッチ監視 | Cron + Dead‑letter Queue |

- ORM の “eager loading / batch loading” 機構を活用し、**N+1** を CI で検知。

---

## 10. ドキュメント

| ドキュメント | 配置 |
|--------------|------|
| ER 図 (Mermaid / dbml) | `docs/er/er_diagram.mmd` |
| マイグレーション README | `<migration_dir>/README.md` |
| クエリガイド | `docs/db/queries/` |
| データ辞書 (英/日) | `docs/db/data_dictionary.md` |

---

## 11. 開発フロー

1. **計画** — RFC / 実装計画書を作成しレビュー。
2. **実装** — スキーマ & マイグレーション追加。
3. **テスト** — ユニット + Integration + 性能。
4. **レビュー** — DDL 差分 / ER 図 / 性能結果。
5. **デプロイ** — CI/CD パイプラインで Apply → Smoke Test。

---

## 12. 運用 & メンテナンス

| 項目 | 頻度 | 具体策 |
|------|------|--------|
| バックアップ | 毎日 | Point‑in‑Time Recovery を有効化 |
| リストア演習 | 四半期 | Staging へ復元し E2E テスト |
| スキーマ監査 | 月次 | 未使用 INDEX / テーブル確認 |
| VACUUM / ANALYZE | 自動 or 週次 | DB エンジンの推奨に従う |

---
