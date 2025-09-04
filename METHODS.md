概要

本リポジトリは、共起分析（Association Rules; FP-growth） と 因果探索（Direct-LiNGAM） を同一データ上で実行し、
	•	共起パターン（無向）
	•	直接因果パターン（有向、残差化あり/なしの比較）

を出力・可視化する再現可能なパイプラインを提供する。


データ
	•	元データ：文部科学省「全国学力・学習状況調査」算数（パブリックユース）
	•	配置例
	•	全件：datasets/02_math_rwNA.csv
	•	群別：datasets/grouped/02_math_rwNA_group{1..4}.csv
	•	問題ID→上位ドメイン対応表：datasets/problem_domain_map.csv
（列例：problem_id,domain_upper）

二値化と NA
	•	各設問を 正解=1 / 不正解=0 に二値化。
	•	共起分析では、正解 (_r) と無回答 (_na) を別アイテムとして扱います
例：MathA_13_r, MathA_13_na
	•	LiNGAMでは原則 NAフラグを使用しません（--use-naflags 0）


能力スコア（Ability）
	•	受検者ごとの正答数を z標準化して ability を定義
（IRT への置換も可能だが、本研究では z を採用）


残差化（LOO：Leave-One-Out Residualization）
	•	各設問 j を目的変数 Y_{ij}、説明変数に ability のみ を用いた 単回帰を実施
	•	受検者 i を学習から外して推定した \hat{Y}{ij}^{(-i)} を用い、残差 R{ij}=Y_{ij}-\hat{Y}_{ij}^{(-i)} を作成
（必要に応じて残差を z 化）
	•	目的：全体能力に起因する“見かけの相関”を除去し、項目固有のズレの連動を捉える

パイプラインは 残差化あり/なし を連続実行（--resid-mode both）。一致度（Jaccard）や代表エッジの比較が可能です。


共起分析（Association Rules）
	•	アルゴリズム：FP-growth（mlxtend.frequent_patterns）
	•	事前フィルタ：$$
|\phi| \ge t_{\phi} \quad \text{or} \quad |\log \mathrm{lift}| \ge t_{LL}.
$$
	•	有意判定：FDR（Benjamini–Hochberg）
	•	可視化：無向ネットワーク（色＝\phiの符号、ラベル＝\phi）

主指標
	•	\phi 係数：2×2表の相関（符号と強さが直感的）
$$
\phi = \frac{ad - bc}{\sqrt{(a+b)(a+c)(b+d)(c+d)}} \, .
$$
	•	lift：独立なら 1。共起の相対強度（稀な事象の過大評価を避けるため \log(lift) を利用）
$$
\mathrm{lift}(A\Rightarrow B)
= \frac{P(A\land B)}{P(A)P(B)}
= \frac{\mathrm{conf}(A\Rightarrow B)}{P(B)} \, ,
\quad
\mathrm{conf}(A\Rightarrow B)=P(B\mid A)=\frac{\mathrm{supp}(A\land B)}{\mathrm{supp}(A)}.
$$


因果探索（Direct-LiNGAM）
	•	入力
	•	残差あり：残差行列 R
	•	残差なし：元の二値行列 X
	•	学習：Direct-LiNGAM（非ガウス・非巡回（DAG）仮定）
	•	安定化：ブートストラップ（例：--n_boot 600 --min-boot-ok 360）、微小 --jitter、--subsample 等
	•	効果量：
$$
\Delta p = P(B=1\mid A=1) - P(B=1\mid A=0).
$$
（Aが成り立つと B の達成確率がどれだけ変わるか）
	•	可視化：有向ネットワーク（色＝\Delta p の符号、ラベル＝\Delta p）
	•	HTML には ノード座標＋エッジ指標（φ/Δp）を JSON 保存する Export ボタンを実装済み


 実行例

1) 全件（残差あり/なしを連続実行）
python run_pipeline.py \
  --raw-csv "datasets/02_math_rwNA.csv" \
  --map-csv "datasets/problem_domain_map.csv" \
  --out-root "runs/pooled_compare_resid3" \
  --pos-top 20 --neg-top 10 --beta-min 0.05 \
  --min-abs-phi 0.14 --min-abs-loglift 0.14 \
  --min-support 0.05 --min-support-abs 20 \
  --min-prevalence 0.02 --max-prevalence 0.98 \
  --standardize z --laplace-alpha 1.0 \
  --use-naflags 0 \
  --n_boot 600 --min-boot-ok 360 --jitter 0.008 --subsample 1.0 \
  --min-deltap 0.08 \
  --resid-mode both \
  --causal-top 30 --causal-sort delta_p \
  --title-prefix "All (Direct)"
2) 群別（G1〜G4）
for i in 1 2 3 4; do
  python run_pipeline.py \
    --raw-csv "datasets/grouped/02_math_rwNA_group${i}.csv" \
    --map-csv "datasets/problem_domain_map.csv" \
    --out-root "runs/grouped_compare_v1/group${i}" \
    --pos-top 20 --neg-top 10 --beta-min 0.05 \
    --min-abs-phi 0.14 --min-abs-loglift 0.14 \
    --min-support 0.05 --min-support-abs 20 \
    --min-prevalence 0.02 --max-prevalence 0.98 \
    --standardize z --laplace-alpha 1.0 \
    --use-naflags 0 \
    --resid-mode both \
    --n_boot 600 --min-boot-ok 360 --jitter 0.008 --subsample 1.0 \
    --min-deltap 0.08 \
    --title-prefix "Group ${i} (Direct, LOO residual)"
done

出力（例）

共起（ARules）
	•	{tag}_arules_all.csv, {tag}_arules_pos.csv, {tag}_arules_neg.csv
	•	{tag}_arules_thresholds.txt（有効件数・しきい値ログ）
	•	{tag}_answer.html（無向ネットワーク；Export で *_nodes_answer.json 出力）

因果（LiNGAM）
	•	{tag}_lingam_edges_ci_{r}.csv（全推定エッジ, r=0 raw / 1 resid）
	•	{tag}_lingam_edges_ci_sig_{r}.csv（有意＆|\Delta p| フィルタ済み）
	•	causal_{tag}_resid{r}.html（有向ネットワーク；Export で nodes_causal_{tag}_resid{r}.json）

画像化（任意）
HTML から保存した nodes JSON を PNG に変換：
python render_from_nodes_json.py \
  --json "runs/jsonfiles_fin/nodes_causal_02_math_group1_resid1.json" \
  --out  "runs/jsonfiles_fin/png/g1_causal_resid1.png" \
  --label-min 0.00

しきい値と選択ロジック
	•	共起：採択 ＝ FDR 有意 かつ 事前フィルタ（|\phi| または |\log(lift)|）通過
	•	因果：採択 ＝ ブート安定性（min_boot_ok）クリア → |\Delta p|\ge しきい値
	•	表示本数は --pos-top / --neg-top / --causal-top で制御

datasets/
  02_math_rwNA.csv
  grouped/02_math_rwNA_group{1..4}.csv
  problem_domain_map.csv
runs/
  <experiment>/
    group{1..4}/
      <tag>_arules_*.csv
      <tag>_lingam_edges_*.csv
      <html / JSON / PNG>
scripts/
  run_pipeline.py
  process_rules.py
  process_lingam.py
  render_from_nodes_json.py

依存関係
	•	Python 3.9+
	•	pandas, numpy, mlxtend, statsmodels, scikit-learn, pyvis, matplotlib, lingam

環境に応じて pip install -r requirements.txt を利用


注意・限界
	•	LiNGAM は 非ガウス・非巡回（DAG）仮定に依存。観察データゆえ、因果仮説は介入での検証が望ましい
	•	しきい値（|\phi|, |\log(lift)|, |\Delta p|）や残差化仕様に対して感度分析を推奨
