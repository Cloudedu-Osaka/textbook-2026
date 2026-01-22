# textbook-2026: 幾何学と計算思考

次年度（2026年度）の授業用テキストブック。

## 学習目標

このテキストブックは、GeoGebra と Python による**双方向学習環境（ggblab）** を活用して、以下を実現します：

1. **幾何学的直感の再発見** — 生徒が古代の数学者とともに定理を「再発見」する過程を体験
2. **計算思考の育成** — 幾何学の依存関係を通じて変数スコープと計算構造を理解
3. **Dual Coding** — ビジュアル（GeoGebra）と記号（Python）の対応関係を明示的に学習

## カリキュラム構造

### Chapter 01: 円を描く — Thales の定理の再創造

**導入的な考え方**:
> 「円を描く」という最も基本的な作図から始め、古代ギリシャの数学者 Thales が発見した定理「直径に対する円周角は直角である」の導出過程を、生徒が自ら再創造する。

**学習フロー** (5段階):
1. **01_intro.ipynb** — なぜ円か？歴史的背景
2. **01_draw_circle.ipynb** — 黒板で円を描く（視覚的直感）
3. **01_geogebra.ipynb** — GeoGebra での操作的な理解
4. **01_protocol.ipynb** — Construction Protocol による構造化
5. **01_verify.ipynb** — Python + ggblab による代数的検証
6. **01_theorem.ipynb** — Thales の定理の導出と証明

### Chapters 02–15: 予定

来年度のカリキュラム構築過程で追加予定。

## 使用技術

- **GeoGebra** — 動的幾何学ソフトウェア
- **Python** — 計算と検証
- **ggblab** — GeoGebra と Python の統合フレームワーク
- **Jupyter** — 対話的学習環境

## はじめに

各章は以下の構造で学習を進めます：

```
[Motivation] → [Visual] → [Structured] → [Computational] → [Proof]
   なぜか？      描く      プロトコル    Python検証        定理導出
```

この流れにより、幾何学が「規則の集合」ではなく、「発見可能な論理体系」であることを学びます。

---

**著者**: Manabu Higashida  
**作成年**: 2026  
**ライセンス**: BSD-3-Clause

