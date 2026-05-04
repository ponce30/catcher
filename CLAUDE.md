# catcher — Claude向け作業ノート

## このリポジトリの目的
NPB捕手データ(「Stat Search Results」CSV)をブラウザでアップロード → 日本語列にマッピング → ソート可能なランキング表示 → `data/` 配下に自動コミット。speed-rankingの兄弟アプリ(走力→捕手版)。

## 技術構成
- **完全静的サイト**(GitHub Pages公開、サーバーロジック無し)
- `index.html` / `style.css` / `app.js` の3ファイル構成
- データ永続化は GitHub Contents API 経由でリポジトリに直接コミット
- 認証はユーザー個人のPAT(localStorage `catcher:pat`)

## 列マッピング(`app.js` の `COLUMN_MAP`)
| src(CSV英) | dst(表示日) | type | agg | 補足 |
|---|---|---|---|---|
| (なし)       | Rk          | num | rank  | POP2B (最速) 昇順で動的付与 |
| Player      | Player      | str | first | グループキー |
| Team        | Team        | str | first | 任意。無くても動作 |
| ARM         | ARM (最速)   | num | max   | prefix一致("ARM (km/h)" 等もOK) |
| POP2B       | POP2B (最速) | num | min   | prefix一致("POP2B (SEC)" 等) |
| EXC         | EXC (最速)   | num | min   | prefix一致 |

### CSVは「生データ」想定
旧仕様(`POP2B (min)` など事前集計済み)から、**1行=1プレー** の生CSVへ変更。`csvToRows` が以下を実施:
1. `POP2B (SEC)` がマイナス・欠損・**`POP_MIN_VALID` (=1.70秒) 以下** の行は計測エラーとして **行ごと除外** (その行のARM/EXC値も捨てる)。文字化けで生まれた異常な高速値や、同一選手として混ざる重複ノイズもこの閾値で取り除く想定
2. Player名にラテン文字(A-Z/a-z)もしくは Unicode replacement char (U+FFFD `�`) が混じる行は文字化け扱いで **行ごと除外**(NPB選手名は日本語のみのため、ラテン文字混入=エクスポート時点で壊れている指標)
3. Player単位でグループ化
4. 各項目の **トップタイム** に集約 (POP2B/EXCは最小値、ARMは最大値、Teamは最初の非空)
5. POP2B(最速)昇順で `Rk` を1始まりで付与

prefix一致のおかげで `POP2B`/`POP2B (SEC)`/`POP2B (min)` どの形式でも動作する。新しい列を追加したい時は `COLUMN_MAP` に entry を足し `agg` を指定。閾値を変えたい時は `POP_MIN_VALID` を編集。

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
