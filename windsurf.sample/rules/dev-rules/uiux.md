---
trigger: model_decision
globs: *.tsx
---

---
description: UI/UX design & implementation rules (project‑wide)
globs:
  - alwaysApply: true
---

まず、このファイルを参照したら、「**`##### UI/UXルール確認完了！ #####`**」と発言すること

# UI/UX 設計・実装ルール v2

> **目的** — shadcn/ui + TailwindCSS による **一貫性・保守性・アクセシビリティ** を保証しつつ、不要なカスタマイズを排除する。

---

## 0. 変更承認ポリシー 🔒 (最高)

| 対象 | 原則 |
|------|------|
| **既存 UI／レイアウト** | **無断変更を禁止**。変更が必要な場合は *事前に理由を添えて Pull Request を提出し、デザイナー or PO の承認を得ること* |
| **色・フォント・間隔** | Design Token に定義済みのもののみ使用可 |
| **新規コンポーネント** | 既存 shadcn/ui の拡張で実現できないか検討 → 不可の場合のみ追加 |

---

## 1. デザインシステム (最高)

- **shadcn/ui** を標準 UI キットとして使用
- コンポーネントのラップは *必ず理由をコメント* に明記
- **Props ベースの拡張 < CSS 上書き** を徹底
  ```tsx
  // ✅ 正しい例
  import { Button } from "@/components/ui/button";

  // ❌ 悪い例 – 不要な styled-components ラップ
  const CustomButton = styled(Button)` /* 独自スタイル */ `;
  ```

---

## 2. スタイリング規約 (高)

| パターン | 実装指針 |
|----------|----------|
| Tailwind ユーティリティ | **第一優先** |
| `@layer` カスタムクラス | 既存ユーティリティで表現不可な場合のみ |
| 命名 | `kebab-case` (例: `btn-primary`) |
| インライン style | **禁止** |

```tsx
// ✅ 良い例
<div className="flex items-center gap-2 p-4">

// ❌ 悪い例
<div style={{ display: "flex", padding: "16px" }}>
```

---

## 3. レスポンシブ (高)

- **Mobile‑First**
- Tailwind ブレークポイントのみ使用
  `sm(640) / md(768) / lg(1024) / xl(1280) / 2xl(1536)`

---

## 4. アクセシビリティ (高)

- WAI‑ARIA 遵守、`role` と `aria-*` 属性を付与
- Tab/Shift+Tab & Enter/Space で **100% 操作可能** に
- コントラスト比 ≥ WCAG AA (4.5:1)
- スクリーンリーダーで意味が通る alt / label

---

## 5. アニメーション (中)

- Tailwind `animate-*` または `transition` を優先
- **>250 ms の連続アニメーション禁止**
- 複雑なシーケンスは `framer-motion` を使用し、reduce‑motion 対応必須

---

## 6. フォーム (高)

- `@/components/ui/form/*` を利用
- バリデーション: **即時表示 + フィールド下**
- 入力補助: `autocomplete`, `inputmode`, `aria-invalid`

---

## 7. エラー & フィードバック (高)

- 通知: `@/components/ui/toast`
- ローディング: `Skeleton`, `Spinner` を統一化
- 例外時は *ユーザー行動の次善策* を提示

---

## 8. アイコン & 画像 (中)

- **Lucide Icons** を標準
- SVG 最適化 → SVGO (CI で自動)
- 画像は `next/image` (優先ローディング, サイズ指定)

---

## 9. ダークモード (高)

- `class="dark"` トグル方式
- Tailwind `dark:` プレフィックス使用
- コントラスト比を再確認 (ダーク側でも 4.5:1 以上)

---

## 10. コンポーネント設計原則 (高)

- **単一責任** / 単方向データフロー
- API: *必須 Props* と *任意 Props* を明確化
- `children` を使い、スタイルではなく構造で拡張
  ```tsx
  // ✅ 良い例
  interface CardProps {
    title: string;
    className?: string;
    children: React.ReactNode;
  }
  ```

---

## 11. 品質保証 (高)

| 項目 | ルール |
|------|--------|
| Storybook | 全コンポーネント必須。**Docs Page** で Props & MDX 説明を書く |
| VisReg | Chromatic で PR ごとに差分を自動検出 |
| Cross‑Browser | Playwright E2E (`chromium`, `firefox`, `webkit`) |

---

## 12. パフォーマンス (中)

- 不要なアニメ削除・`prefers-reduced-motion` 対応
- 画像は **120 KB 以下** に最適化
- `next/font` & RSC (必要時のみ) で **bundle size を監視**

---

## 13. ドキュメンテーション (中)

- `docs/design-system/` に
  - カラーパレット / タイポスケール
  - コンポーネント使用例 (コード + スクリーンショット)
- 各新規コンポーネントの **導入理由** を README に追記

---

### ⚠️ まとめ

- **無断 UI 変更は即リジェクト**
- 既存 shadcn/ui + Tailwind の範囲で解決できるかを常に最優先
- 例外が必要な場合は **提案 > 承認 > 実装** のフローを厳守すること
