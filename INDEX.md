# INDEX — agent-atelier-data

最終更新：2026-11-14
状態：現行
プロジェクト状態：進行中（記事の公開にあわせてパッケージを追加する）
現行を確認する共有先：GitHub public repo `ksaomitsutsummm/agent-atelier-data`、ブランチ `main`

## この repo は何か

The Agent Atelier の記事で引用した数値の裏付けとなる、公開データパッケージの置き場。各パッケージに、事前登録した実験プロトコル、per-image スコア、判定記録、評価グリッド等を含む。パッケージの一覧は README.md。

## 役割と正本の対応

| 役割 | 正本 |
|---|---|
| 入口 | 本ファイル（パッケージ一覧は README.md） |
| 現状・制作記録 | 正本 repo `ksaomitsutsummm/agent-atelier` の `drafts/<記事>/`（本 repo は公開側の写し） |
| 決定 | 正本 repo の `ops/decisions.md` |
| 引き継ぎ | 正本 repo の `ops/handover.md` |

## 注意

- 原寸 PNG の全量など、容量の大きい一次素材は本 repo に入れない（記事001の原寸240枚・約267MBは未収録。正本側の保管場所は各 drafts の README を参照）。
- 学習データ等の第三者素材を含む場合は、著作者・ライセンス表示をパッケージ内に含める（記事003のパッケージが実施済み）。
