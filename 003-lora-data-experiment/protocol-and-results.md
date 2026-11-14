# Protocol and results — 003 LoRA data-side experiment

This file is the frozen pre-registered protocol (registered 2026-11-13, SHA-256 `53f5456d98ce6c1ab46472f640490263587726104292bd7cd8a13ba921f65e07`) followed by the lab's final report, reproduced without edits to the numbers.

---

# PROTOCOL — レッサーパンダLoRAのデータ側改善・事前登録

## 登録と目的
登録日時: 2026-11-13T03:25:22.390044+00:00 UTC。新規データ収集・学習・評価画像生成の前に登録。登録後は本文を変更しない。SHA256をPROTOCOL.sha256に保存し、以後の実行で照合する。逸脱・接続障害はSTATUS.mdに追記する。
現行8枚・均一キャプション・60stepのLoRAに対し、構図を分散し画像別キャプションを付けた24枚の新LoRAが未使用seedで改善支持条件を満たすか検証する。公開・投稿・配布はしない。Kaggle/Modalの非公開計算に必要な転送のみを実施する。

## データセット
写真24枚、Wikimedia CommonsのファイルページおよびAPIで著作者とCC0/CC BY/CC BY-SAの明示を確認する。曖昧なライセンス、無断転載、生成画像、イラストは採用しない。全件に出典ページURL・画像取得URL・著作者・ライセンス名とURL・取得日時・原画像と処理画像のSHA256をdata/SOURCES.jsonに記録する。
元8枚は含めない。元画像の木上静止構図と同一撮影系列への依存を避け、拡張した資料から概念を学習できるかを見るため。従って旧セットへの単純な16枚追加の効果ではなく、全体を組み替える介入である。
24枚は主分類を重複させず、地上移動4、能動的な木登り4、食事4、顔・上半身の寄り4、地上の全身静止4、複数個体4とする。複数の属性がある場合は主分類を1つ決め、別途補助タグを記録する。木上で単独静止する全身構図は主分類に設けない。全身が収まる画像を合計8枚以上、異なる著作者を6名以上、同著作者4枚以下とする。連写・同じ画像のクロップ違い・視覚的な近重複は除外する。既知の評価に特化した雪・海辺・川の画像は収集条件に加えず、それらの背景画像は除外する。
検索結果を行動・構図・品質・権利情報だけで選ぶ。モデル評価を見て選び直さない。候補の却下理由を記録する。最低短辺512pxの原画像または高解像度サムネイルを取得、EXIF回転適用、RGB化、被写体を残す正方形クロップ、512×512にLanczos縮小、JPEG quality92で保存。歪ませるリサイズ、合成、生成補完、拡張増殖、反転は行わない。クロップ座標は原画像に対して記録し、全身分類で足・尾が切れる写真は採用しない。24枚が揃わない場合は学習せずSTATUSに代替案を提示し、独断で枚数や分類を変更しない。

## キャプション
全画像に対応する.txtを作り、先頭をrpandaに統一。見える個体数、動作、構図、背景を短い英文で記述し、見えない属性・撮影地の推測・品質宣伝タグを入れない。data/CAPTIONS.mdにも一覧化する。例（実採用画像の内容に合わせて確定）：
- rpanda, a photograph of a red panda walking on grass, full body and striped tail visible
- rpanda, a photograph of a red panda climbing a tree trunk, side view
- rpanda, a photograph of a red panda eating bamboo leaves, seated on the ground
- rpanda, a close-up photograph of a red panda face, green foliage in the background
- rpanda, a photograph of two red pandas standing together on the ground
句のshuffleは旧実験同様有効、ただしkeep_tokens=1でトリガーを先頭に維持する。キャプションdropoutなし。

## 学習条件と交絡
1ラン、最終180stepのみ。中間バリアントなし、強度探索なし、途中checkpointの選別なし。途中失敗時は障害を記録し、学習開始後の再実行は自動で行わない。
- kohya sd-scripts commit cb70b96858477e716a2096292ab8aab433277f38 をcheckoutして照合。
- SD1.5 v1-5-pruned-emaonly.safetensors、SHA256 6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa。
- networks.lora、rank=8、alpha=8、UNetとtext encoder両方を学習。learning_rate=1e-4（UNet/TE共通）、AdamW、constant scheduler、warmupなし。
- batch=2、gradient accumulation=1、seed=42、resolution=512,512、fp16学習・保存、cache_latents、sdpa、workers=2。
- 24枚×5 repeats、max_train_steps=180。旧8枚×5 repeats×3epochs=60stepと比べ、画像当たりの平均露出は双方15回（360/24対120/8）。学習量の総量は3倍。optimizer等の明記しない引数は固定commitの既定値を使い、実コマンドと重みmetadataに保存する。
- 出力 rpanda_data24.safetensors、最終重みSHAとログを回収。
変更点は、8→24枚・全画像入替、均一→内容別caption、中心→被写体保持cropと明示Lanczos、keep_tokens既定0→1、60→180step、epoch指定→step指定。その他上記条件は元の学習を維持。前回のUNet-only300step実験とは異なりTEも元条件通り学習する。
これは「データ側パッケージとしての複合介入であり、個別要因の分離はできない」。データだけの因果効果、枚数だけの効果、キャプションだけの効果とは主張しない。180stepは露出量を揃えるために固定し、結果に応じて増減しない。
Kaggle専用非公開notebookでSaveAndRunAll、enableGpu=true、machineShape=NvidiaTeslaT4。投入前後にget_accelerator_quotaを実測保存。全体1800秒、学習subprocess1200秒上限とし、接続結果不明時は状態を照合して重複投入しない。

## 評価と選択／確認分離
選択seed=[314159, 271828]、確認seed=[161803, 141421]。過去の1234/5678/9012/3456/13579/24680と学習42を一切評価に用いない。各phaseは10文×2seed×3条件=60枚、計120枚。
比較はbase（LoRAノードなし）、current（元LoRA SHA256 98c024391dd8e76fa6c854baf69bed04e0d627dce82ba5674d1f81d91808532b、UNet/TE強度1/1）、new（最終180step、強度1/1）のみ。
単一候補のため選択phaseは探索的診断・候補SHA凍結に使用し、成績による候補変更や確認中止をしない。選択の全盲検採点と数値集計後に候補SHAを記録し、それから確認を生成する。確認結果のみを改善主張の採否に用いる。選択結果との平均混合はしない。
推論は前回同様、512×512、batch1、20steps、CFG7、euler/normal、denoise1、負文 `blurry, low quality, distorted`、clip skip追加なし。ComfyUI v0.3.68 commit 265adad858e1f31b66cd3523a02b16f5d34ced52、torch2.5.1+cu121、T4。モデルSHA・依存環境・workflow・画像SHAを保存して前回と照合する。
CLIPはopenai/clip-vit-base-patch32 revision 3d74acf9a28c67741b2f4f2ea7635f0aaf6f0268、transformers4.46.3、CPU float32、正規化埋め込みの内積。logitやsoftmaxではない。評価文は先頭の `rpanda, ` のみ除去し、他は原文のまま。対照・trigger除去文は変更なし。

## 固定プロンプト（前回config原文）
1. [near_training] `rpanda, a photograph of a red panda in a bamboo forest`
2. [near_training] `rpanda, a photograph of a red panda resting on a tree branch`
3. [near_training] `rpanda, a close-up photograph of a red panda face, detailed fur`
4. [generalization] `rpanda, a photograph of a red panda walking through snow`
5. [generalization] `rpanda, a watercolor painting of a red panda beside a river`
6. [generalization] `rpanda, a photograph of a red panda on a sandy beach at sunset`
7. [control] `a photograph of a golden retriever in a garden`
8. [control] `a photograph of a mountain landscape with a lake`
9. [trigger_ablation] `a photograph of a red panda in a bamboo forest`
10. [trigger_ablation] `a photograph of a red panda walking through snow`

## 盲検目視と開示手順
各phaseで全60画像にランダムな匿名IDを付け、対応表とCLIP数値を表示しないシートを作る。promptは提示する。条件ラベル、workflow/PNG metadata、ファイル名からの推定を避ける。採点CSVを保存しSHA256と日時をロックした後だけ対応表・数値を開く。ロック後の採点変更は禁止。各画像の指示適合、形態、画質を0=悪い/1=混在/2=良いで採点、所見も残す。風景の形態は明確な構造破綻がなければ2。
採点者は本エージェントであり、独立した人間評価ではない。実装者と同じで画風から条件を推測する余地があり、完全な盲検を保証しない。提示サイズを記録する。
系統的な重い目視退行を事前に定義：確認の近接＋汎化12対応ケースについて、newの形態または指示適合が0、かつ比較対象が1以上である退行が、currentまたはbaseのいずれかに対して2ケース以上ある場合。どちらかに該当すれば不支持。画質0の同種退行も同じ規則で判定する。中程度の退行や対照群の破綻も全件報告する。

## 支持条件と集計
確認phaseで以下を全て満たす場合のみ「本事前固定評価で改善を支持」とする。
1. 主群（近接写真3＋汎化3、2seed、12対応ケース）CLIP平均差が、new-current、new-baseともに厳密に+0.001を超える。
2. 各確認seedの主群6ケース平均差が、両比較とも厳密に0を超える。
3. 対照2文×2seedのnew-current平均差が−0.01以上。
4. 上記の系統的な重い目視退行がない。
不合格なら「改善未確認」。技術的未完了は「実験未完了・判定不能」とし、不支持と混同しない。都合のよい群や選択phaseで判定を覆さず、改善版は作らない。
各phaseで条件別・群別CLIP平均、同一prompt/seed対応差、seed別、勝敗件数、目視各軸平均と全個別所見を報告する。主群の参考95%区間は6promptをclusterとし2seed平均差を5000回bootstrap、乱数8841、percentile法。小標本で統計的確証とはしない。全体平均・trigger除去群は診断のみ、主指標へ二重加重しない。

## 成果物と終了
PROTOCOLとSHA、STATUS、出典/画像/caption、学習code/log/最終重みSHA、評価code/画像/metadata/CLIP/盲検採点/lock/開示集計、REPORTをラボに保存する。取得日時・実行日時・障害・逸脱を記録。資格情報と署名付きURLはファイル/ログに保存しない。Modalはバッチ実行し終了時に関連appのstopped/tasks0を確認・記録、常駐を残さない。他者の無関係なジョブを停止しない。
REPORTは結論、全支持条件との照合、前回との対応比較、限界、失敗、Kaggle quota差、Modal実測時間（請求時間と区別）、脚色しない記事材料をまとめる。外部公開は対象外。


---

# レッサーパンダLoRA・データ側改善実験

**事前固定の確認評価で改善を支持した。** 選択結果だけで判断せず、確認用シード161803/141421の対応比較で判定した。

## 事前登録と方法

PROTOCOL.mdを収集・学習・評価前に固定。SHA256 `53f5456d98ce6c1ab46472f640490263587726104292bd7cd8a13ba921f65e07`。元8枚を含まないCommons写真24枚、地上移動・木登り・食事・顔の寄り・地上全身静止・複数個体を各4枚。撮影者22名、同一撮影者最大3枚。全身がフレーム内に収まる写真9枚（自然な遮蔽を含む）。出典・著作者・ライセンスはdata/SOURCES.json、元ページ/API保存はdata/provenance/、キャプション一覧はdata/CAPTIONS.md。

正方形の被写体保持crop、512px、Lanczos、JPEG quality92。個別captionはrpandaを維持。UNetとTEをともに学習しrank/alpha8、AdamW、lr1e-4 constant、batch2、seed42、fp16、keep_tokens1、最終180stepのみ。24枚×5repeat、360画像提示で平均15露出。旧8枚60stepも平均15露出だが、画像入替・枚数・caption・crop・shuffle制御・総学習量の複合介入で、個別要因の効果を分離できない。中間checkpoint選別・強度探索・学習済み結果に基づく再学習は行っていない。

学習器commit `cb70b96858477e716a2096292ab8aab433277f38`。SD1.5ベースSHA `6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa`。新重み `artifacts/training/rpanda_data24.safetensors` SHA256 `a526ff7f648bf536129f2418b4e97345fe31e7dfff58e0bc2f2fdcbeb36932a7`。全24画像/captionのローカル固定SHAとリモート学習入力の一致を確認。

比較はbase（LoRAノードなし）、current（8枚60step、強度1/1）、new（24枚180step、強度1/1）。選択314159/271828、確認161803/141421、同じ10文、各60枚の計120枚。単一候補で選択成績にかかわらず確認へ進む設計。主群は近接写真3＋汎化3の12対応ケース。対照2・trigger除去2は別集計。

推論512×512、20steps、CFG7、Euler/normal、denoise1、負文 `blurry, low quality, distorted`、ComfyUI v0.3.68、torch2.5.1+cu121、T4。CLIPはopenai/clip-vit-base-patch32 revision `3d74acf9a28c67741b2f4f2ea7635f0aaf6f0268`、transformers4.46.3、CPU float32の正規化埋め込み内積。先頭rpandaのみ評価文から除去。

## 選択（探索的診断）

| 条件 | 主CLIP | 現行比 | base比 | 適合/2 | 形態/2 | 画質/2 | 合計/6 |
|---|---|---|---|---|---|---|---|
| base | 0.357648 | 0.012410 | 0.000000 | 1.833333 | 1.583333 | 2.000000 | 5.416667 |
| current | 0.345238 | 0.000000 | -0.012410 | 1.416667 | 1.916667 | 2.000000 | 5.333333 |
| new | 0.350198 | 0.004959 | -0.007451 | 1.916667 | 1.666667 | 1.833333 | 5.416667 |

| newのseed | 主CLIP | 現行比 | base比 |
|---|---|---|---|
| 314159 | 0.345307 | 0.013531 | -0.011410 |
| 271828 | 0.355089 | -0.003612 | -0.003492 |

| newの群 | n | CLIP | 現行比 | base比 |
|---|---|---|---|---|
| primary | 12 | 0.350198 | 0.004959 | -0.007451 |
| all | 20 | 0.343443 | 0.002675 | -0.005337 |
| control | 4 | 0.316477 | -0.002004 | -0.003400 |
| generalization | 6 | 0.357346 | 0.013775 | -0.000481 |
| near_training | 6 | 0.343049 | -0.003856 | -0.014420 |
| trigger_ablation | 4 | 0.350143 | 0.000499 | -0.000933 |

主群の現行比参考95%区間 [-0.005663, 0.016978]、base比 [-0.014420, -0.001102]。6promptをclusterにして2seed平均差を5000回bootstrap、乱数8841。小標本の参考区間で統計的確証ではない。

盲検60件採点ロック: 2026-11-13T10:17:45.078932+00:00、SHA `3fa94acdbea07dccd83ed2f815a2a86581e3c2ca2a836cacae555f77cc19ddc1`。条件名とCLIPを隠した396pxシートで本エージェントが採点し、保存・SHAロック後に開示。独立した人間評価ではない。個別所見は各phaseのblind/ratings.csv、対応差はvisual_pairs.json。

## 確認（主判定）

| 条件 | 主CLIP | 現行比 | base比 | 適合/2 | 形態/2 | 画質/2 | 合計/6 |
|---|---|---|---|---|---|---|---|
| base | 0.354228 | 0.004452 | 0.000000 | 1.833333 | 1.750000 | 2.000000 | 5.583333 |
| current | 0.349777 | 0.000000 | -0.004452 | 1.833333 | 1.833333 | 2.000000 | 5.666667 |
| new | 0.365686 | 0.015910 | 0.011458 | 1.750000 | 1.916667 | 1.916667 | 5.583333 |

| newのseed | 主CLIP | 現行比 | base比 |
|---|---|---|---|
| 161803 | 0.360144 | 0.014725 | 0.010477 |
| 141421 | 0.371228 | 0.017094 | 0.012439 |

| newの群 | n | CLIP | 現行比 | base比 |
|---|---|---|---|---|
| primary | 12 | 0.365686 | 0.015910 | 0.011458 |
| all | 20 | 0.358566 | 0.011121 | 0.008395 |
| control | 4 | 0.332485 | -0.002028 | 0.001046 |
| generalization | 6 | 0.379034 | 0.026158 | 0.009721 |
| near_training | 6 | 0.352339 | 0.005661 | 0.013195 |
| trigger_ablation | 4 | 0.363286 | 0.009901 | 0.006554 |

主群の現行比参考95%区間 [0.004347, 0.025703]、base比 [0.006600, 0.016652]。6promptをclusterにして2seed平均差を5000回bootstrap、乱数8841。小標本の参考区間で統計的確証ではない。

盲検60件採点ロック: 2026-11-13T10:26:51.586119+00:00、SHA `ddcc81f9ac72ee90f71db7904630c784835c131dc3a8b95955ac21c196cc5af7`。条件名とCLIPを隠した396pxシートで本エージェントが採点し、保存・SHAロック後に開示。独立した人間評価ではない。個別所見は各phaseのblind/ratings.csv、対応差はvisual_pairs.json。

## 支持条件との照合

| 条件 | 合否 |
|---|---|
| primary_both_gt_001 | 合格 |
| each_seed_both_positive | 合格 |
| control_current_gte_minus_01 | 合格 |
| no_systematic_severe_visual_regression | 合格 |

主CLIPの両比較が厳密に+0.001超、各seedで両比較が正、対照の現行比が−0.01以上、系統的な重い目視退行なしを全て要求した。形態または適合0で比較対象1以上の退行が同じ比較に2ケース以上、あるいは画質0の同種退行が2ケース以上なら不合格。実際の件数: `{"base": {"severe_content_or_anatomy": 0, "severe_quality": 0}, "current": {"severe_content_or_anatomy": 0, "severe_quality": 0}}`。

## 障害・資源・限界

最初の埋込ZIP方式はKaggleのソース1MB制限で保存拒否（実ランなし）。version1はデータ準備でraw画像SHA不一致となり75秒で例外終了、学習開始なし。元URLのローカル再取得24件は全件一致し、配信経路差の原因は未確定。データを変えず該当画像の固定JPEGを直接転送し、その他も最終処理SHA一致を必須にした。詳細と各版はSTATUSとartifacts/preparation-v1。Kaggle状態APIのRUNNING残留と最終summary/予約0の不整合も記録した。

出力一覧APIの署名付きURLを一度ログ保存・表示してしまい、発見直後にローカルログから全削除してファイル名だけへ置換した。これは非保存方針への逸脱。以後ダウンロードURLはメモリ内だけで使用した。外部への投稿・公開・配布は行っていない。

成功学習のスクリプト全体 288.9秒、学習サブプロセス 190.082秒。Kaggle quotaとModalの実測・終了確認はlogsの原記録に基づき末尾へ追記する。アプリ寿命・生成処理時間・GPU請求時間は別物で、請求秒数/料金は未取得。

前回のUNet-only300stepは現行比−0.010369、base比−0.008855で改善未確認だった。今回とはseedが異なるため絶対CLIP平均同士の差を改善量と解釈せず、それぞれのphase内で現行/baseとの対応差を見る。

未確認: 独立した人間の評価、多数seed/広いprompt/別checkpointへの一般化、学習画像の記憶・個体再現、単一要因の因果効果。CLIPと目視の相違も併記し、改善未確認を「効果がない証明」とは扱わない。

## 記事にできる点

- 写真の枚数だけでなく、構図・caption・crop・出典を事前に規定し、学習に渡したバイトを照合する手順。
- 画像当たり露出量を揃えても総学習量は変わるため、データ側パッケージの複合介入として解釈する必要。
- 選択と確認の対応差、CLIPと形態/指示適合を分け、陰性結果もそのまま残す実験規律。

## 資源実測と終了確認（監督追記 2026-11-13）

- Kaggle quota（GPU）: 実験全体の開始前532.440秒 → v1準備失敗後861.575秒 → v2成功後910.664秒（いずれも予約0、総枠108000秒、refresh 2026-11-14T00:00:00Z）。v2の差分は約49.1秒で、スクリプト全体288.9秒・学習190.1秒とは一致しない。Kaggle側のquota計量方法は未確認であり、差分を学習時間と解釈しない。原記録はlogs/quota-*.json。
- Modal: 選択生成のlocal wall 307.1秒、確認生成319.5秒（各60枚、生成自体は1枚約4.5秒）。CLIP採点のremote elapsedは選択68.8秒、確認8.0秒（metrics.json記録値）。終了時にapp listでastra-lora-data-labの全インスタンスがstopped・tasks 0を確認（2026-11-13T10:27Z以降）。常駐は残っていない。請求額は未取得で、上記は実測時間であり料金ではない。
- 前任セッションは2026-11-13T03:55:57Zに接続障害で停止し、v2の結果確認が約6時間遅れた。再開監督がセッション状態COMPLETEとrun_summaryを直接回収して継続した。再投入は行っていない。

## 目視の解釈（確認フェーズ主群）

- 主群の目視合計（/6）は new 5.583、base 5.583、current 5.667 とほぼ同等。newは形態（1.917）が3条件で最も高く、適合（1.750）と画質（1.917）はcurrent（1.833/2.000）をわずかに下回った。支持判定は事前登録どおりCLIP主指標と「系統的な重い退行なし」の組合せで決まっており、目視は退行ガードとして合格（重い退行0件）だった。目視でnewが全面的に優位だったとは主張しない。
- 個別の弱点として、newを含む複数条件で「竹に抱きつく前足が手のように大きく指が長い」画像が確認フェーズで2件観察された（条件帰属はvisual_ratings_unblinded.csvとkeyで追える）。0点の形態は確認主群のnewにはなかった。
- 採点者は実装と同一の本エージェントで、独立した人間評価ではない。完全な盲検は保証されない（PROTOCOLの限界どおり）。
