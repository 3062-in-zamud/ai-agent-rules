---
trigger: glob
description: "Node.js implementation rules"
globs: ["**/*.tsx", "**/*.ts", "**/*.js"]
---

---
description: Node.js (TypeScript) universal development rules & guidelines — framework‑agnostic and future‑proof
globs: ["**/*.tsx", "**/*.ts", "**/*.js"]
ruleType: "Glob"
---

まず、このファイルを参照したら、この **`##### Node.jsルール確認完了！ #####`** を発言すること

# Node.js (TypeScript) 開発ルール

> **目的** — Express・Fastify・NestJS・Hono・Lambda など **どのスタックでも通用** し、
> **5 年後でも腐らない** “基本原則” を明文化する。
> **変更** — 例外や新ベストプラクティスは **RFC → Review → ルール更新** という流れを厳守。

---

## 1. ランタイム & パッケージ方針

| 項目 | ルール |
|------|--------|
| Node.js | **Active LTS** 以上を `engines.node` に指定（固定しない） |
| Package Manager | `pnpm` 推奨。lockfile を必ずコミット |
| SemVer | ライブラリ `^`、CLI/Tooling `~`、内部 pkg は exact (`1.2.3`) |
| Scripts | `npm run start:dev`, `build`, `test`, `lint`, `typecheck`, `migrate` 等 **共通化** |
| ENV 管理 | `.env.example` に **全キー** を記載しドキュメント化 |

---

## 2. TypeScript コンパイラ & 設定

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "strict": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "useUnknownInCatchVariables": true,
    "importsNotUsedAsValues": "error",
    "moduleResolution": "NodeNext",
    "verbatimModuleSyntax": true,
    "esModuleInterop": false,
    "resolveJsonModule": true,
    "skipLibCheck": true,          // CI 高速化、型精度は `types` で補完
    "outDir": "dist"
  }
}
```

- Prod build は `tsc -p tsconfig.build.json` または `esbuild` で **ESM 出力**。
- `paths` / `baseUrl` によるエイリアスは **最小限**（深い依存を避けるため）。

---

## 3. コーディング規約 & 静的解析

| ツール | 設定 |
|--------|------|
| **ESLint** | `@typescript-eslint`, `eslint-plugin-import`, `eslint-plugin-security` |
| **Prettier** | `printWidth 100`, `singleQuote true`, `arrowParens avoid` |
| **Husky + lint‑staged** | `pre-commit`: lint → typecheck → test --bail |
| **Naming** | `camelCase` (var/fn), `PascalCase` (class/type), `SCREAMING_SNAKE` (const) |
| **Import順** | Node built‑ins → external → internal → relative。Enforced via `eslint-plugin-import` |

CI は `eslint --max-warnings 0` で警告ゼロを強制。

---

## 4. エラーハンドリング指針

1. **ドメインエラー型** — `class AppError extends Error { status: number; cause?: Error }`
2. **Boundary での一括捕捉** — HTTP/GraphQL middleware ／ Lambda wrapper 等で `AppError`→ JSON 成形。
3. **非同期例外** は `@typescript-eslint/no-floating-promises` で検出。
4. **Process Level** — `unhandledRejection` / `uncaughtException` をログ後に exit(1) し、supervisor (PM2/systemd/Docker restart) に委ねる。

---

## 5. セキュリティ & コンフィグ

| 項目 | ベストプラクティス |
|------|-------------------|
| Secrets | `dotenv-expand` → runtime injection。**コードリポジトリに置かない** |
| HTTP Headers | `helmet` (Express) / `fastify-helmet` 等 |
| Input Validation | `zod` / `joi` / `class-validator`。I/O は **100% スキーマ** で検証 |
| Dependency Audit | `pnpm audit --prod --audit-level=high` (CI fail) + Renovate |

---

## 6. ロギング & 観測性

| レイヤ | 推奨ツール |
|--------|-----------|
| Structured Log | `pino` (`level: info`, env var override) |
| Tracing | OpenTelemetry SDK (`@opentelemetry/sdk-node`) |
| Metrics | `prom-client` for Prometheus |
| Correlation ID | middleware で `x-request-id` 付与・伝播 |

---

## 7. テスト戦略 (MECE)

| 種類 | ツール | 目的 |
|------|-------|------|
| Unit | **Vitest** or Jest | 単機能 / 分岐網羅 |
| Integration | Supertest / Fastify Inject | DB・外部サービス (test doubles) |
| Contract | `pact-js` | Producer & Consumer |
| E2E | Playwright / Cypress (if FE) | システム信頼線 |
| Perf/Load | k6 / Artillery | SLA 検証 |

- カバレッジ目標: **80 % Lines, 70 % Branches**。`nyc` / built‑in coverage で CI fail 未満。
- テストデータは **Fixture ファイル + Faker**。環境汚染を避けるため **独立トランザクション** or **Docker Compose** テスト DB を使用。

---

## 8. データアクセス

- ORM or Query Builder は自由（Prisma, Drizzle, Knex, MikroORM…）。
- **Repository or DataSource パターン** で抽象化し、テスト時にインメモリ実装へ差替え可能に。
- DB 設計ルールは **database-rules.mdc** を参照。

---

## 9. パフォーマンス基準

| 指標 | 目標 |
|------|------|
| Cold Start (Lambda) | < 2 s |
| p95 Latency (API) | < 300 ms |
| Memory Footprint | < 300 MB / instance |
| Bundle Size (Edge) | < 2 MB |

> **計測** — k6 + OpenTelemetry + Prometheus → Grafana Dashboard に可視化。

---

## 10. CI/CD パイプライン (参考ステップ)

1. **Install** — `pnpm i --frozen-lockfile`
2. **Static** — ESLint, Prettier Check, `tsc --noEmit`, Audit
3. **Test** — Unit → Integration → Contract
4. **Build** — `tsc` or `esbuild` to `./dist`
5. **Docker** — Multi‑stage build (`node:slim`) → `docker scan`
6. **Deploy** — GitHub Actions → Registry → K8s / ECS / CloudRun / Lambda
7. **Smoke** — `/healthz` + log verify; Rollback on failure

---

## 11. ドキュメント & コミュニケーション

| ドキュメント | 内容 |
|--------------|------|
| **README** | Setup / Scripts / ENV / Run & Test |
| **CONTRIBUTING.md** | branch model, commit style (Conventional Commits) |
| **ADR** | 技術決定ログ (`docs/adr/nnn-title.md`) |
| **API Spec** | `openapi.yaml` or GraphQL SDL auto‑generated |
| **Runbook** | On‑call 手順・SLI/SLO |

Markdown ファイルは **markdown‑lint** で CI チェック。Broken links は fail。

---

## 12. ルール変更プロセス

1. **提案** — `implementation_plans/YYYYMMDD_rfc_<topic>/` で背景・案・代替策を整理。
2. **レビュー** — TechLead + QA + PO がレビュー → 承認。
3. **反映** — 本ルールを PR で更新。
4. **リファクタ** — 必要に応じ既存コードを `@todo.md` タスクで段階的に適用。

---
