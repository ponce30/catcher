# catcher — Claude向け作業ノート

## このリポジトリの目的
NPB捕手データ(「Stat Search Results」CSV)をブラウザでアップロード → 日本語列にマッピング → ソート可能なランキング表示 → `data/` 配下に自動コミット。speed-rankingの兄弟アプリ(走力→捕手版)。

## 技術構成
- **完全静的サイト**(GitHub Pages公開、サーバーロジック無し)
- `index.html` / `style.css` / `app.js` の3ファイル構成
- データ永続化は GitHub Contents API 経由でリポジトリに直接コミット
- 認証はユーザー個人のPAT(localStorage `catcher:pat`)

## 列マッピング(`app.js` の `COLUMN_MAP`)
| src(CSV英) | dst(表示日) | type |
|---|---|---|
| Rk          | Rk          | num |
| Player      | Player      | str |
| EXC (min)   | EXC (最速)   | num |
| ARM (max)   | ARM (最速)   | num |
| POP2B (min) | POP2B (最速) | num |

新しい列を追加・削除したい時はこの配列を編集。`(min)` は時間が短い=最速、`(max)` は速度が大きい=最速、で「最速」に統一。

## データフォルダ
- `data/` 配下に `YYYY-MM-DD_HHmmss.csv` の名前で自動保存
- アップロードのオリジナルCSV(英語ヘッダのまま)を保存している
- 履歴一覧は Contents API で取得し、最新を初期表示する

## PATが必要な操作
- 履歴一覧取得 (`GET /contents/data`) — public repoなら無くても動くが、rate limit緩和のため設定推奨
- 履歴ファイル取得 (`GET /contents/data/...`) — 同上
- CSVコミット (`PUT /contents/data/...`) — 必須

## 編集時の注意
- ユーザーが「catcher編集」と言ったら `cd "/c/Users/ckg19/OneDrive/デスクトップ/catcher"` → `git pull` → 編集 → commit/push
- commit時は `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>` を付ける
- user.email は `141467642+ponce30@users.noreply.github.com`

## ブラウザでの動作確認
- index.html を直接ブラウザで開いてもCORSで動かない(GitHub APIは動くが)
- ローカル確認は `python -m http.server 8000` 等で起動して `http://localhost:8000`
- preview設定は `../.claude/launch.json` の "catcher Static Preview" を使う
