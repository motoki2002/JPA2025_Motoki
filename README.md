# JPA2025_Motoki

# Problem-to-Problem Dependencies in Elementary Math: Association Rules & Causal Discovery

本リポジトリは、全国学力・学習状況調査（算数・パブリックユースデータ）を用いて、  
**設問間の関係性**をアソシエーションルール（ARules）と因果発見（LiNGAM）で分析した結果を公開する。  
再現用の**コードは含まない**が、使用パラメータ・出力スキーマ・成果物を完全に開示する。

## 概要
- **目的**：設問間の共起関係（φ, lift）と、能力を統制した上での**直接的因果候補（Δp）**を比較し、群（学力層）ごとの違いを明らかにする。
- **新規性**：学習ログではなく"設問—設問"レベルの関係に着目し、  
  ①人間に解釈可能なARules、②残差化（LOO）×Direct-LiNGAMの因果候補、を**同一フレームで対比**。
- **主な発見**：残差化後、**高学力群は疎/低学力群は密**な因果ネットワーク。  
  つまずきの**局所連鎖（ボトルネック）**が低学力群で顕在化。

## データ
- 出典：文部科学省「全国学力・学習状況調査」パブリックユースデータ（算数）https://www.mext.go.jp/a_menu/shotou/gakuryoku-chousa/sonota/1404609.htm
- 取得方法：公開サイトからダウンロードし、正誤データのみを抽出し、`datasets/02_math_rwNA.csv` へ配置

## 成果物（抜粋）
- 因果ネットワーク（PyVis HTML / PNG）
- ARules ネットワーク（PyVis HTML / PNG）
- nodes系JSON（HTMLから保存：座標・エッジ指標）
- 集計図表（Jaccard一致度、Δp分布、正/負エッジ比、NA比など）

## 解析パラメータ（固定票）
ARules:
- itemset_scheme: `r+na`
- |φ| 事前フィルタ: `tφ = 0.14`
- |log-lift| 事前フィルタ: `tLL = 0.14`
- min_support: `0.05`（abs: `20`）
- 多重比較補正: Benjamini–Hochberg（FDR 0.05）
- NAを結論側に禁止: 解析によってON/OFF（標準はOFF）

LiNGAM:
- 残差化：**LOO線形回帰**（能力 = 正答合計のz）、各設問を能力に回帰 → 残差XをDirect-LiNGAMへ投入
- 標準化：z
- ブート：`n_boot=600`, `min_boot_ok=360`, `jitter=0.008`, `subsample=1.0`
- 有意化：95% CI（bootstrap）、効果量指標：`Δp`、閾値 `|Δp| ≥ 0.08`
- 可視化上限：`top=30`（|Δp|優先）

> 詳細は `parameters.yml` と `METHODS.md` 参照。

## 出力スキーマ（ダイジェスト）
- **ARules CSV**：`A_id, B_id, phi, lift, confidence, support, p_value, p_fdr, fdr_sig, domA, domB, ...`
- **LiNGAM CSV**：`u, v, beta, delta_p, ci_lo, ci_hi, boot_ok, sig, dom_u, dom_v, ...`
- **nodes JSON（HTMLエクスポート）**  
  - `title`: 図タイトル  
  - `nodes`: `[ {id, x, y}, ... ]`  
  - `edges`:  
    - ARules: `{u, v, phi}`（無向）  
    - LiNGAM: `{u, v, delta_p}`（有向）

> フル版は `SCHEMA.md` を参照。

## リポジトリ構成（例）
html/               # PyVis HTML（公開用）
figures/              # PNG図
runs/jsonfiles_fin/   # nodes系JSON（HTMLから保存）
tables/               # 集計CSV
parameters.yml
METHODS.md
SCHEMA.md
README.md
