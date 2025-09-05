# Causal & Association Graph Gallery (JPA 2025)

このページは GitHub Pages で公開される **インタラクティブな可視化集** です。  

**Repository:** [github.com/motoki2002/JPA2025_Motoki](https://github.com/motoki2002/JPA2025_Motoki)

こちらのGithubからAssociationRulesやLiNGAMの結果の詳細csvファイルなどを閲覧することができます(下記の個別リンクから飛ぶことも可能です)。

---

## 1) Association Rules (ARules)

- **Pooled (All)**
  - [ARules graph — All](graphs/02_math_answer.html)

- **By Group**
  - [Group 1 — ARules](graphs/02_math_group1_answer.html)
  - [Group 2 — ARules](graphs/02_math_group2_answer.html)
  - [Group 3 — ARules](graphs/02_math_group3_answer.html)
  - [Group 4 — ARules](graphs/02_math_group4_answer.html)

> ラベルは基本 **φ（phi）** を表示。事前フィルタは `|phi| ≥ tφ` **OR** `|log(lift)| ≥ tLL`。  
> ルール採択は FDR 有意（必要に応じて |φ| 下限で安定化）。

---

## 2) Causal Networks (LiNGAM)

- **Pooled (All)**
  - [LiNGAM — residualized (LOO)](graphs/causal_02_math_resid1.html)
  - [LiNGAM — non-residualized](graphs/causal_02_math_resid0.html)

- **By Group**
  - **Residualized (LOO)**
    - [G1](graphs/causal_02_math_group1_resid1.html) ・
      [G2](graphs/causal_02_math_group2_resid1.html) ・
      [G3](graphs/causal_02_math_group3_resid1.html) ・
      [G4](graphs/causal_02_math_group4_resid1.html)
  - **Non-residualized**
    - [G1](graphs/causal_02_math_group1_resid0.html) ・
      [G2](graphs/causal_02_math_group2_resid0.html) ・
      [G3](graphs/causal_02_math_group3_resid0.html) ・
      [G4](graphs/causal_02_math_group4_resid0.html)

> エッジ色は **Δp** の符号（正=赤/負=青）、ラベルは **Δp**。  
> タイトル右上の **Export positions** ボタンでノード座標とエッジ指標（Δp/φ）を JSON 保存できます。

---

## 3) Static Figures & Tables

- 図（PNG）：[figures/](../figures/) (Github)
- 追加の表/CSV：[outputs/](../outputs/)(Github)

---

## 4) Methods & Notes

- **Methods（手法の概要）**：[METHODS.md](../METHODS.md)
- **ポスター PDF**（学会発表用）：[poster](../poster)(Github)
