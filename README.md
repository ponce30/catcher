# catcher

NPB捕手ポップタイム/送球データのアップロード→ランキング表示・自動保存ツール。

## 公開URL
https://ponce30.github.io/catcher/

## 必要なCSV列(「Stat Search Results」のフィルターで以下を選択して書き出す)
| 元(英) | 表示(日) | 内容 |
|---|---|---|
| Rk | Rk | ランク |
| Player | Player | 選手名 |
| EXC (min) | EXC (最速) | 捕球→送球の交換時間 最速(秒) |
| ARM (max) | ARM (最速) | 送球初速 最大(km/h) |
| POP2B (min) | POP2B (最速) | 二塁送球ポップタイム 最速(秒) |

CSVヘッダはこの英語名と完全一致させること。並び順は問わない。

## 使い方
1. 「Stat Search Results」のCSV(英語ヘッダ)をドロップ
2. 自動で日本語列にマッピングされ、テーブル表示(初期ソート: POP2B 最速 昇順)
3. 列ヘッダクリックで昇順↔降順ソート
4. アップロードしたCSVは `data/` 配下に自動コミット(履歴として残る)
5. 「履歴」セレクトから過去のCSVを再表示可能

## GitHub PAT設定(初回のみ)
1. https://github.com/settings/tokens で `repo` scope付きFine-grained or Classic PATを発行
2. アプリ内「⚙️ GitHub PAT設定」を開きペースト→保存
3. ブラウザのlocalStorageに保存される(他人には漏れない)

## 構成
- 静的サイト(GitHub Pages)
- アップロード→GitHub Contents APIで `data/YYYY-MM-DD_HHmmss.csv` にコミット
- 履歴一覧もContents APIで取得
