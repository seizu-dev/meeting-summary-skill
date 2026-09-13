# Obsidian Vault との統合（オプション）

議事録を Obsidian Vault で管理している場合、[../SKILL.md](../SKILL.md) の Step 2（事前情報の収集）と Step 7（保存）を Vault と連携させられる。

この文書は特定の Vault 構成に依存しない書き方にしてある。フォルダ名・タグ名・プラグインは自分の構成に読み替えること。

## Obsidian CLI

Vault 内の検索には Obsidian CLI を使う（全文検索が効き、`grep` より意図に近い結果が得られる）。

### 所在と有効化

**Obsidian CLI は Obsidian アプリ同梱の `obsidian` コマンドそのもの。** サードパーティ製の別バイナリではないので、`obs` や `obsidian-cli` のような名前で探しても見つからない。

- PowerShell では `Obsidian.com` に解決される
- PATH に無い場合、実体は Windows なら `$LOCALAPPDATA/Programs/Obsidian/Obsidian.com`
- 実体を直接叩いて `Command line interface is not enabled.` が返る場合、**CLI 機能が無効になっている**。Obsidian の Settings > General > Advanced で有効化する

**有効化は PC ごとに必要**で、Vault の同期内容には含まれない。複数の PC で同じ Vault を使う場合、どの PC でも使える前提を置かないこと。使えない PC では `grep` / `rg` で代替する。

### 主なサブコマンド

```bash
obsidian search query=<text> vault=<name> format=json
obsidian "search:context" query=<text> path=<folder> format=json
obsidian read path="<folder>/<note>.md"
```

その他: `tags` / `properties` / `backlinks` / `links` / `orphans` / `bases` / `base:query`

`vault=<name>` で Vault を指定、`file=` は名前解決（ウィキリンク相当）、`path=` は完全パス。

## Step 2 の拡張: Vault から事前情報を集める

会議に対応するタスクノート・案件ノートが Vault にあるなら、そこから事前情報を自動収集できる。

1. **対象ノートを特定する。** 引数（タイトル・ファイル名・ウィキリンク）から Obsidian CLI で検索する。見つからない・複数候補がある場合はユーザーに確認する。
2. **frontmatter を読む。** 日時・参加者・関連ノートへのリンクなど。
3. **本文を読む。** 事前メモ・確認したい項目・サブタスク。
4. **リンク先を辿る。** `related` などでリンクされた案件ノート、同じ案件の過去の議事録。

**1つの案件の情報は複数フォルダに分散している前提で横断的に探すこと。** 案件ノート・議事録・タスクがそれぞれ別フォルダにある構成は珍しくない。片方だけ見て「情報が無い」と判断すると、既知の事実を「未確認」と書いてしまう。

集めた内容から固有名詞リストを作る（[../SKILL.md](../SKILL.md) の Step 2）。

## Step 7 の拡張: Vault へ保存する

- 保存は **Obsidian CLI ではなく Write ツール / コマンドラインで行う**（CLI に書き込み系サブコマンドは無い）
- 保存先とファイル名は Vault の規約に合わせる（例: `12_Meetings/YYYY/MM-DD <タイトル>.md`）
- 議事録用のテンプレートがあれば、その見出し構成・粒度に揃える
- **ファイル名・見出しに `#` を含めない。** Obsidian のウィキリンク `[[...]]` では `#` が見出しアンカーとして解釈されるため、リンクが壊れる（`C#` → `CSharp` のように置き換える）
- **Markdown リンク `[テキスト](URL)` のテキスト部分に `[` `]` を含めない。** コードスパン内でもエスケープが効かず、`]` の位置でリンクが途切れる
- ノート冒頭に H1 見出しを書かない構成であれば、それに従う（ファイル名がタイトルとして表示される Vault では重複になる）

frontmatter の例:

```yaml
---
date: YYYY-MM-DD
tags: [meeting]
participants: [ ... ]
related: ["[[対象タスクノート]]"]
---
```

## タスク管理プラグインとの連携

TaskNotes のようなタスク管理プラグインを使っている場合、議事録作成後にタスク側も更新する。

1. 対象タスクノートの `related` に、作成した議事録へのリンクが無ければ追記する
2. タスクが完了扱いになる性質のもの（面談実施など）なら、ユーザー確認のうえ状態を更新する

**完了にする際は、ステータスと完了日を必ずセットで更新すること。** ステータスだけ変えて完了日を入れないと、Dataview / TaskNotes の完了タスク集計に拾われない。

また、Dataview のクエリが `scheduled` を基準に集計している場合、`due` だけを設定したタスクは期限切れ扱いで紛れ込む（`date(null) < this.date` が true になるため）。タスクを作るときは `scheduled` を必ず設定する。

## Git 同期

Vault を Git で同期している場合、保存後にコミットする。

```bash
git add <議事録フォルダ>/ <タスクフォルダ>/
git commit -m "docs(meetings): YYYY-MM-DD <タイトル> の議事録を追加"
git push origin main
```

Obsidian の Git プラグインで自動同期している場合は、手動コミットと競合しないよう自動同期の間隔に注意する。
