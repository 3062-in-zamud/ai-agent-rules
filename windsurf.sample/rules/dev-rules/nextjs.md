---
trigger: glob
description: "Next.js implementation rules"
globs: ["**/*.tsx", "**/*.ts", "**/*.js"]
---

---
description: "Next.js best‑practice rules (project‑wide)"
globs: ["**/*.tsx", "**/*.ts", "**/*.js"]
ruleType: "Glob"
---

まず、このファイルを参照したら、「**`##### Next.jsルール確認完了！ #####`**」と発言すること

# Next.js 実装ルール

> **目的** — App Router + Server Components を前提に “DX と保守性” を最大化しつつ、
> **実装計画 (`implementation_plans/`)・TODO (`@todo.md`)・ルール自体** を “常に最新版” へ柔軟に進化させる。

---

## 0. 変更承認ポリシー 🔒 (最高)

- **既存ページ / API / 公開コンポーネントの破壊的変更を禁止**
  - 変更提案は **実装計画書** (RFC 形式可) を起票し、PO 承認を得る。
  - `@todo.md` に `[ ]` タスクを追加してリンク。
- **本ルール自体の改訂**
  1. `implementation_plans/YYYYMMDD_nextjs_rule_update/` を作成し、改訂理由・差分を記述。
  2. PR に “`rules-update`” ラベルを付与。
  3. PO + TechLead がレビュー後、マージで発効。
  4. `@todo.md` に `[x]` 完了を記録。

---

## 1. ディレクトリ & ルーティング

```
src/
├── app/
│   ├── (marketing)/ …        # 公開ページ (Server Component)
│   ├── (dashboard)/ …        # 認証必須エリア
│   └── api/ …                # API Routes (POST/PATCH/PUT/DELETE のみ)
├── components/
│   ├── common/ …             # 汎用
│   ├── features/ …           # 機能別
│   └── layouts/ …            # page 単位 Layout
├── hooks/ …                  # Client‑only hooks
├── lib/ …                    # utils / types / constants
├── dal/ …                    # Data Access Layer (Prisma 等)
└── public/ …                 # 静的アセット
```

### 命名

| 種類 | ファイル名 |
|------|------------|
| Page | `page.tsx` |
| Layout | `layout.tsx` |
| Error Boundary | `error.tsx` |
| Loading UI | `loading.tsx` |
| 404 | `not-found.tsx` |

---

## 2. コンポーネント指針 & **`use client` 判定フローチャート**

| 種別 | 原則 |
|------|------|
| **Server Component (デフォルト)** | データ取得 / SEO / 初期描画に JS 不要 |
| **Client Component** | 下記フローで “YES” になった場合のみ |

```
┌─> React Hook使用? ────────────┐
│                               │
│   ┌─> ブラウザAPI必要? ──┐    │
│   │                      │    │
│   │   ┌─> onClick 等     │ NO │
│ NO┤   │     のイベント? ─┤────┤→ ❶
│   │   └───────────────┘    │
│   │                          │
│   └─────── YES ─────────────┘
│
└─────── YES (いずれか) ──────────> **Client** + 先頭に `use client`
```

❶ **クライアントでも JS が不要** な単純リンク等は *Server Component* 推奨。

> **メモ** — “迷ったら Server” を鉄則とし、パフォーマンス計測結果で再評価。

---

## 3. API Route ルール

- `app/api/<resource>/route.ts` で実装。
- 許可メソッド: **POST / PATCH / PUT / DELETE** のみ。**GET API は基本禁止**（フェッチは Server‑Side）。
- Prisma 等は **DAL 層経由**。
- 認証・バリデーションを必ず実装。
- 各新 API は必ず `@todo.md` にタスクを追加し、実装計画書を添付。

```ts
// POST /api/articles
import { NextResponse } from "next/server";
export async function POST(req: Request) {
  try {
    const body = await req.json();
    const article = await dal.article.create(body);
    return NextResponse.json(article, { status: 201 });
  } catch (e) {
    return NextResponse.json({ error: "Internal Error" }, { status: 500 });
  }
}
```

---

## 4. データ取得 & キャッシュ

| シナリオ                       | fetch オプション                            |
|--------------------------------|---------------------------------------------|
| ISR (60 m)                     | `{ next: { revalidate: 3600 } }`            |
| ユーザー固有 / 最新必須        | `{ cache: "no-store" }`                     |
| 完全静的（ビルド時取得）       | デフォルト                                  |

---

## 5. UI/UX & shadcn/ui 連携

- コンポーネントは **UI/UX ルール.mdc** の規定を厳守。
- 新 UI 追加時は **実装計画書にスクリーンショット** を貼付し、デザイナー承認を得る。

---

## 6. 型安全 & Lint

- `tsconfig.json` で `"strict": true` を必須とする。
- ESLint / Prettier 設定は **monorepo 共有** を使用。
- API 入出力は `zod` か `ts-pattern` でスキーマバリデーション。

---

## 7. セキュリティ

- 機密値は `.env` に保存し、クライアント公開が必要なもののみ `NEXT_PUBLIC_` プレフィックス採用。
- すべての API Route に **認可ガード** を実装。
- `next.config.js` で **CSP / Security Headers** を設定。

---

## 8. パフォーマンス

- 画像は **`next/image`**、フォントは **`next/font`** を使用。
- 外部 Script は **`<Script strategy="lazyOnload" />`** で遅延読込。
- Lighthouse スコア < 90 の PR は `@todo.md` に `[!]` ブロッカータスクを起票。

---

## 9. エラーハンドラ & ローディング

- 各 `app/(group)/` 直下に `error.tsx` と `loading.tsx` を配置。
- Suspense で粒度細かく分割し、部分ローディングを実現。
- 例外時は **ユーザーに次善策** (再試行ボタン等) を提示。

---

## 10. デプロイ & CI

- **`NEXT_TELEMETRY_DISABLED=1`** を CI 環境に設定。
- edge‑runtime バンドルサイズが **2 MB 超** えたら CI で警告を出す。
- 本番環境の ENV 変更時は **実装計画書** + **週次レビュー** で共有。

---

## 11. メンテナンス

- Dependabot PR を **週次** でまとめてマージ。
- `prisma migrate` 実行時は **ER 図自動更新タスク** もセットで登録。
- **依存パッケージ / Next.js のメジャーアップグレードが出たら**
  - `implementation_plans/YYYYMMDD_upgrade_next_<version>/` を起票し、互換性チェックリストを使用。
- ルールと実装の乖離を検知したら **即 RFC** を作成してフローに従い更新。

---

## 12. 実装計画 / TODO 連携フロー

1. **新ページや API** を作る → `implementation_plans/YYYYMMDD_<title>/` を作成し設計を書く。
2. 同内容を **`@todo.md`** に `[ ]` タスクでリンク。
3. 作業中は Progress ログを追加し、`@todo.md` のステータスを `[~]` に更新。
4. PR マージ後 `[x]` に変更し **Definition of Done** を満たすことを確認。

> *このフローを守らない PR はレビュアーが即リジェクトしてください。*

---
