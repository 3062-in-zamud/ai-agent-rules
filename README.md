# AI エージェントルール設定

このリポジトリには、AI エージェント（Cursor と Windsurf）用のルール設定サンプルが含まれています。

## 使用方法

各サンプルディレクトリをリネームして使用します。以下の名前変更例に従ってください：

- `cursor.sample` → `.cursor`
- `windsurf.sample` → `.windsurf`

### Cursor の設定

1. `cursor.sample` ディレクトリを `.cursor` にリネームします
2. ルール設定は `rules` フォルダ内にあります
   - `globals.mdc`: 基本的なグローバルルール
   - `globals_detailed.mdc`: 詳細なグローバルルール
   - `process-gates.mdc`: プロセスゲートの設定
   - `dev-rules/`: 開発用のルール

### Windsurf の設定

1. `windsurf.sample` ディレクトリを `.windsurf` にリネームします
2. ルール設定は `rules` フォルダ内にあります
   - `globals.md`: 基本的なグローバルルール
   - `globals_detailed.md`: 詳細なグローバルルール
   - `process-gates.md`: プロセスゲートの設定
   - `dev-rules/`: 開発用のルール

## 注意事項

- リネーム後のディレクトリ名はドットから始まるため、隠しディレクトリとなります
- ファイルの内容を必要に応じて編集してカスタマイズできます
