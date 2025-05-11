---
trigger: glob
description: "Python implementation rules"
globs: ["**/*.py"]
---

---
description: "Python universal development rules & guidelines — framework‑agnostic, future‑proof"
globs: ["**/*.py"]
ruleType: "Glob"
---

まず、このファイルを参照したら、この **`##### Pythonルール確認完了！ #####`** を発言すること

# Python 開発ルール v1.0

> **目的** — Django / FastAPI / Flask / Serverless など **どのスタックでも通用** し、
> 5 年後でも腐らない “Python プロジェクトの衛生基盤” を提供する。
> **変更** — 例外や新ベストプラクティスは **RFC → Review → ルール更新** を必須とする。

---

## 1. バージョン & 環境

| 項目 | ルール |
|------|--------|
| Python | `pyproject.toml` の `[tool.poetry.dependencies] python` に **Active LTS (≥ 3.12)** を範囲指定 |
| 仮想環境 | `poetry` 推奨 (`poetry.lock` をコミット)。既存 venv/pipenv を尊重し移行は任意 |
| OS | CI は Linux (ubuntu‑latest) 基準。macOS/Windows は smoke‑test のみに |

---

## 2. コーディング規約 & 静的解析

| ツール | ルール |
|-------|--------|
| **PEP 8** | 準拠 (エンフォース via flake8 + `flake8-bugbear`) |
| **Black** | `line-length 100`, `target-version py312` — 自動整形 |
| **isort** | Black モードで import 整列 |
| **Flake8** | `--max-complexity 10 --max-line-length 100`。CI で warning 0 強制 |
| **mypy** | `strict = True` (gradual OK)。CI で type error が 0 になるまで無視コメント可 |

`pre-commit` で black → isort → flake8 → mypy を自動実行。

---

## 3. 依存管理 & セキュリティ

| 項目 | ベストプラクティス |
|------|-------------------|
| SemVer | ライブラリ `^`, 開発ツール `~` |
| 依存の可視化 | `poetry show --tree` を `docs/deps.md` に出力 (CI 更新チェック) |
| 脆弱性スキャン | `pip-audit` (CI fail >= high) |
| Secrets | `.env` or `dotenv`。Git にコミット厳禁 (pre‑commit hook) |

---

## 4. 型ヒント & ドキュメンテーション

- **PEP 484 / 604** 型注釈を必ず付与 (public API)。
- 複雑な構造は **TypedDict / dataclass(frozen=True)** を使用。
- Docstring は **PEP 257** & Google Style (`pydocstyle`, `darglint` で lint)。
- 自動 API docs (FastAPI / Sphinx) で型情報を活かす。

---

## 5. エラーハンドリング & ロギング

| 項目 | ガイドライン |
|------|--------------|
| 例外階層 | `class AppError(Exception)` を基底にドメイン別サブクラス |
| 例外メッセージ | 英語、文末ピリオドなし、Actionable |
| 非同期 | `async` / `await` + `anyio`。未使用 coroutine は `# noqa: E800` 禁止 |
| ロギング | `structlog` JSON logger (`timestamp`, `event`, `level`, `service`) |
| 重大例外 | `sys.excepthook` でログ後 `exit(1)`。プロセス管理は Docker / systemd 任せ |

---

## 6. I/O & バリデーション

- 入出力スキーマは **pydantic v2** または `dataclasses-jsonschema`。
- 外部 API コールは **10 s timeout** & **再試行** (`tenacity`)。
- SQL インジェクション回避: パラメータバインディングのみ使用。

---

## 7. テスト戦略

| レイヤ | ツール | カバレッジ目標 |
|--------|-------|--------------|
| Unit | **pytest** (`pytest-cov`) | 85 % Lines |
| Integration | `pytest` + Docker Compose service | 70 % |
| Contract | `schemathesis` (OpenAPI), `pact-python` | 100 % critical endpoints |
| E2E | `playwright-python` / robotframework | Happy path |
| 性能 | `pytest-benchmark` | p95 ∆ < 10 % |

- Fixture: `pytest‑factoryboy`, `faker`.
- Coverage < 80 % で CI fail。`[# pragma: no cover]` はレビュー必須。

---

## 8. DB & OR/M

- ORM 選択自由 (SQLAlchemy 2.0, Tortoise‑ORM, Django ORM)。
- **Repository / Service 層** で抽象化し、ドメイン層を ORM 非依存に。
- スキーマ設計は **database-rules.mdc** 準拠。
- マイグレーションは `alembic` / `yoyo‑migrations` / Django Migrations。`--sql` dry‑run を PR 添付。

---

## 9. パフォーマンス & 観測性

| 項目 | 推奨 |
|------|-----|
| 非同期 I/O | `asyncio + uvloop` or Trio / anyio |
| Caching | `aioredis` / `fastapi-cache` / functools lru_cache |
| トレーシング | OpenTelemetry (`opentelemetry-sdk`, OTLP exporter) |
| Metrics | `prometheus-client` — default `/metrics` |
| Profiling | `py-spy` / `flamegraph` in CI on demand |

---

## 10. CI/CD (GitHub Actions例)

1. **Setup** — `actions/setup-python@v4` + Poetry cache
2. **Lint** — Black diff, isort diff, flake8, mypy
3. **Test** — pytest + coverage
4. **Security** — pip‑audit, bandit (`--severity-level high`)
5. **Build** — `poetry build` or Docker multi‑stage (`python:slim`)
6. **Deploy** — Container Registry → Kubernetes / Lambda / Cloud Run
7. **Smoke** — `/healthz` probe + log check、失敗で自動 Rollback

---

## 11. ドキュメント & ナレッジ

| Doc | 必須内容 |
|-----|---------|
| README | Setup / Scripts / ENV / Run & Test |
| CONTRIBUTING | Branch, commit style, pre‑commit hook |
| ADR | 技術決定ログ (`docs/adr/NNN-<title>.md`) |
| API Spec | `openapi.yaml` / `asyncapi.yaml` |
| RUNBOOK | On‑call 手順 / SLO / Troubleshoot |

CI に **markdown‑lint**, **link‑checker** を追加。

---

## 12. ルール更新プロセス

1. **提案** — `implementation_plans/YYYYMMDD_rfc_python_rule_<topic>/` を作成。
2. **レビュー** — TechLead + QA + PO で合意。
3. **反映** — この `.mdc` を更新、Docs を同期。
4. **リファクタ** — 既存コードは `@todo.md` で段階的に適用。

---
