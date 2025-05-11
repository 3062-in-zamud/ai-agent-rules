---
trigger: always_on
description: "Implementation Process Gates"
globs: ["**/*"]
---

---
description: Implementation‑Flow Gate — block any code‑changing action until a plan is approved and enforce progress logs
globs:
  - "**/*"
ruleType: Always
---

# Implementation Process Gates

このルールは **実装計画の承認** と **進捗ログ生成** を強制し、品質を自律的に担保するための Gate を定義します。

---

## 0. ルール読了確認（必須）
- このルールを読み終えたら、以下のように発言すること：
```
**`##### 進捗管理ルール確認完了！`**
```
- この確認が行われない限り、実装計画書や進捗ログの生成は行ってはならない。

---

## 1. トリガートークン

| イベント          | 必須トークン                | 追加パラメータ             |
| ------------- | --------------------- | ------------------- |
| **実装計画 承認**   | `APPROVED_IMPL_PLAN:` | 承認した計画ファイルまたは PR 番号 |
| **承認却下（要修正）** | `REVISE_IMPL_PLAN:`   | 修正理由 (20 文字以上)      |

### 1.1 トークン検出前の禁止行動

* `APPROVED_IMPL_PLAN:` がチャットに出る **前に** 以下のツールを呼んではならない：

  * `canmore.create_textdoc` / `canmore.update_textdoc`
  * `python_user_visible`

---

## 2. 進捗ログ必須アクション

1. **マイルストーン完了直後** に以下を連続して実行すること：

   1. `python_user_visible` で変更ファイル一覧を DataFrame 化し可視化
   2. `canmore.create_textdoc` で `progress/YYYYMMDD_progress_<n>.md` を生成
2. 生成後、チャットに次のメッセージを送る：

   ```
   ##### Progress log created
   ```

---

## 3. セルフテスト

* 本ルールを満たさない場合、直後の応答で以下を含むメッセージを出力し、**いかなるファイル更新も行わない**。

  ```
  ### GATE VIOLATION
  ```

---

## 4. 例

```text
User> 実装計画OKです。APPROVED_IMPL_PLAN:docs/plan_v2.md
Assistant> ##### Gate Passed — approved plan docs/plan_v2.md
```

---

## 5. 適用範囲

* globs: `**/*` — リポジトリ全体に適用
* ruleType: `Agent Requested` — 実装計画関連のシーンでのみ自動付与

---

(End of process‑gates.mdc)
