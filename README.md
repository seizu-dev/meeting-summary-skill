# meeting-summary-skill

ローカルに保存された会議の録音ファイルを Gemini Notebook（2026年7月16日に NotebookLM から改名）に渡して文字起こし・要約させ、Markdown のミーティングノートを作成する [Claude Code](https://claude.com/claude-code) 用スキルです。

自動文字起こしが使えない状況 — 無料プランに落ちた会議レコーダー、Windows のサウンドレコーダーや IC レコーダーでの自前録音、Zoom のローカル録画など — で**録音ファイルだけが手元に残っている**ケースを想定しています。

録音ソースは選びません。ffmpeg が読める音声・動画ファイルであれば、パスを渡すだけで処理できます。tl;dv と Windows サウンドレコーダーについては、保存先・命名規約・実測値を [references/](references/) にまとめてあります。

## できること

1. 録音ディレクトリから対象ファイルを特定する（コンテナに記録された**録音開始時刻**を会議の予定時刻と照合。曖昧なときは候補を提示して確認）
2. ffmpeg で Gemini Notebook が受け付ける形式（mp3 / モノラル / 16kHz）へ変換する
3. 会議の目的・参加者・関連資料から**固有名詞リスト**を作り、音声と一緒に Gemini Notebook へ渡す
4. アジェンダ / 議事内容 / 決定事項 / ネクストアクション / 要フォロー事項の構成で要約させる
5. ドラフトをレビューにかけたうえで Markdown として保存する

日本語の会議を前提に作られています。プロンプトも日本語で書かれています。

## 前提条件

| 項目 | 内容 |
| --- | --- |
| Claude Code | スキルの実行環境 |
| ffmpeg | 必須。録音ファイルの変換に使う |
| Gemini Notebook MCP | **必須。** 文字起こしと要約を Gemini Notebook に任せる |
| Gemini Notebook アカウント | notebook 100件 / 1 notebook あたり source 50件が上限 |

Gemini Notebook MCP は claude.ai のコネクタとして接続する構成で検証しています。MCP ツール名のプレフィクスは接続方法によって変わります（claude.ai 経由なら `mcp__claude_ai_notebooklm__*`）。**コネクタ名・ツール名は改名に追随しておらず `notebooklm` のまま**なので、接続先を探すときは旧称で探してください。

ローカルの Whisper などで代替する経路は**検証していない**ため、このスキルには含めていません。

## インストール

ユーザー全体で使う場合:

```bash
git clone https://github.com/seizu-dev/meeting-summary-skill.git ~/.claude/skills/meeting-summary
```

特定のプロジェクトでのみ使う場合:

```bash
git clone https://github.com/seizu-dev/meeting-summary-skill.git <project>/.claude/skills/meeting-summary
```

`SKILL.md` と `references/` がセットで配置されていれば動きます。

## 使い方

```
/meeting-summary <録音ファイルのパス>
```

パスを省略した場合は、録音の保存先を尋ねたうえで候補を新しい順に列挙します。

処理中はユーザーへの確認が2回入ります。

- **ファイル選択** — 会議の予定時刻の前後3時間に候補が1件だけのときを除き、自動では決めません
- **ドラフトレビュー** — Gemini Notebook の回答をそのまま保存せず、必ず内容を確認してもらいます

所要時間の目安は、54分の録音で**変換とアップロードに20秒前後、Gemini Notebook の処理待ちで1〜2分**です（実測値は [references/tldv.md](references/tldv.md)）。

## 既知の制限

- **日本語の固有名詞・同音異義語の誤認識が多い。** 「受託開発」が「自宅開発」、「スカウト」が「スカート」になるなど、日本語として意味の通る別の語に化けるため読んでも気づきにくいです。事前情報として固有名詞リストを渡す手順を組み込んでいるのはこのためで、それでも裏取りは必要です
- **元の録音ファイルを直接アップロードできない。** `.webm` は HTTP 400 で拒否されるため、ffmpeg 変換は省略できません
- **アップロード用の署名 URL は約15分で失効する。** 変換を先に済ませてから URL を取得する順序になっています
- **録音ツールによっては録音長が2つの値を持つ。** プレーヤー表示と `ffprobe` の値が食い違うケースの詳細は [references/tldv.md](references/tldv.md) を参照してください

## リポジトリ構成

```
.
├── SKILL.md                        # スキル本体（録音ソース非依存・ノートアプリ非依存）
└── references/
    ├── tldv.md                     # tl;dv のローカル録音に関する実測知見
    ├── windows-sound-recorder.md   # Windows サウンドレコーダーの録音に関する実測知見
    └── obsidian.md                 # Obsidian Vault と統合する場合のオプション手順
```

`references/` の内容はスキルの動作に必須ではありません。該当する環境で使う場合のみ参照されます。

## ライセンス

MIT License. 詳細は [LICENSE](LICENSE) を参照してください。
