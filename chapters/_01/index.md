# Chapter 01: 円を描く — Thales の定理の再創造

## 概要

古代ギリシャの数学者 Thales（タレス）は、円に関する最初の幾何学的定理を発見しました：

> **Thales の定理**: 円の直径の上にある円周角は、常に直角（90度）である。

このチャプターでは、その発見過程を再現し、以下の学習到達目標を達成します：

1. **幾何学的直感** — なぜこの定理は真なのか？ビジュアルな理解
2. **構造化** — この構築を層ごとに分解し、依存関係を明示
3. **計算的検証** — Python で数値的・代数的に証明
4. **概念転移** — 幾何学と計算構造の対応関係の理解

## 歴史的背景と学問的意義

### 直径（diameter）と古代ギリシア幾何学の転機

直径 (diameter) は、古代ギリシアの幾何学において極めて重要な役割を果たしました。Thales が発見した「円の直径の上にある円周角は常に直角である」という数学的事実は、単なる定理に留まりません。それは、円という図形の根本的な対称性の認識であり、**幾何学における測定と証明の方法論の確立**を意味していました。

### 古典的な発展と停滞：Apollonius から Dandelin へ

Thales の発見は、後世の数学者たちへ大きな影響を与えました。特に **Apollonius** は、円錐曲線論の中で、直径がコニック・セクション（楕円、放物線、双曲線）における評価軸であることを認識し、膨大な論証を積み重ねました。しかし、Apollonius の問題設定には本質的な限界がありました。

円錐の頂点を通る回転軸を中心とした断面は、圓錐曲線論の本質を捉えています。しかし Apollonius が思索した派生的な断面（回転軸に対して傾いた任意の平面による断面）は、真の情報をもたらさず、むしろ複雑性のみを増加させました。結果として、Apollonius の理論は膨大な論証の集積となり、後世、数学的権威として祭り上げられる一方で、その理論体系は新たな学問的進展へは繋がりませんでした。古典的幾何学は、やがて袋小路へ向かうことになります。

やがて、19世紀に **Gaspard Monge** と **Dandelín** は、円錐曲線論を根本から再構築しました。**Dandelin の球**（Dandelin spheres）の概念により、円錐に内接する二つの球から円錐曲線論を再構築するという優雅な方法が提示されたのです。

Dandelin による再構築の本質は、一見すると三次元的に見えますが、実は**二次元の断面に帰結**しています。その証明の心臓部は、二つの円の**共通接線**（common tangent）の性質にあります。具体的には：

1. 円錐に内接する二つの球（上球 $S_1$、下球 $S_2$）を考える
2. 切断平面が球と接する点を $F_1$、$F_2$ とする（焦点）
3. 切断平面上の任意の点 $P$ から母線を引くと、$P$ における距離 $|PF_1| + |PF_2|$ は常に一定
4. この不変性は、二つの円（各球と円錐の交線である円）の共通接線性に由来する

この完備な証明は、幾何学的直感と代数的厳密性の統一をもたらしますが、残念ながら、教育的な文脈では語られることがほとんどありません。本カリキュラムは、まさにこのような「忘れられた真理の再発見」をねらいとしています。

**参考**: 共通接線の概念や円への接線の性質については、[Tangent lines to circles - Wikipedia](https://en.wikipedia.org/wiki/Tangent_lines_to_circles) を参照してください。

### 科学的飛躍への転換点：Kepler による楕円の幾何学の発見

古代幾何学が静止する中、科学の歴史は大きく転換します。その転換点は、**Johannes Kepler** による楕円軌道の物理学への応用（惑星運動の法則）です。これは単なる応用ではなく、幾何学そのものの本質的な再発見をもたらしました。

Kepler の最大の洞察は、**楕円の焦点（foci）**の概念の発見にあります。楕円とは、「二つの焦点からの距離の和が常に一定である点の集合」として定義されます。この定義は一見すると直観的ではありません。なぜなら、円の場合、中心からの距離が一定ですが、楕円の場合、焦点という二つの特別な点からの距離の和という、一段階高い複雑性をもつからです。

Kepler が見抜いた幾何学的洞察は、Dandelin の球による二つの円の接線性にさかのぼります。具体的には：

- **外的共通接線** (external common tangent) と **内的共通接線** (internal common tangent) の非対称な外観
- この一見非対称な構造が、実は楕円の焦点に関する**対称的性質**を隠し持っていること
- その対称性が、惑星軌道という物理現象の中で初めて顕在化すること

二つの円の接線の性質から出発し、その複雑な非対称性の中に隠された対称性を見抜く必要があります。この見方こそが、古典幾何学から現代数学への転換を可能にしたのです。

千年以上の時間、円錐曲線論は象牙の塔の中に留まっていましたが、Kepler による楕円の焦点の発見と天体軌道への応用こそが、**科学的進展の本当の転機**となったのです。

### このカリキュラムの位置付け

本カリキュラムが Thales の定理を再度学ぶ意義は、単に歴史的な追悼ではなく、以下の三点にあります：

1. **抽象化の力**：幾何学的事実を記号と層構造で表現する
2. **検証の方法論**：コンピュータによる自動検証と数学的証明の対応
3. **現代への連続性**：古典的真理がいかに現代の計算科学へ継承されているかの認識

「なぜこの定理は真か」への複数の答え方を学ぶことで、数学的思考そのものの深さを体験することが、今日の学習者にとって必要とされているのです。

#### 双対的対話：GeoGebra と Python の相補的役割

本講義の特徴は、**GeoGebra と Python の双対的対話利用**にあります。これらのツールは単なる計算機ではなく、異なる認識的役割を担います：

- **GeoGebra**：動的操作による幾何学的直感の形成
  - 点をドラッグして「何が変わり、何が変わらないか」を観察
  - 視覚的な規則性の発見と仮説形成
  
- **Python**：その直感に対する論理的導入と形式化
  - 数値計算による性質の検証
  - 依存関係のグラフ化と層構造の明示
  - 自動検証による確実性の確保

この二つの方法が対話することで、学習者は「観察」から「検証」を経て「証明」へと段階的に上昇できます。古代ギリシアの幾何学者たちが時間をかけて達成した発見のプロセスを、現代の学習環境で圧縮された形で再現することが、本カリキュラムの核心なのです。

## 学習フロー

### 1️⃣ **01_intro.ipynb** — なぜ「円」なのか？

**学習内容**:
- 古代ギリシャにおける幾何学の意義
- Thales の生涯と時代背景
- なぜ「直径に対する円周角」を問題にしたのか？
- 現代のカリキュラムでこれを学ぶ意味

**期待される疑問**:
- 「円っていつ発明されたの？」
- 「円周角ってなに？」
- 「直角であることは、何に使うの？」

---

### 2️⃣ **01_draw_circle.ipynb** — 黒板で円を描く

**学習内容**:
- 黒板で手作業で円を描く方法
- 「完全な円」を描くことの困難性
- 中心からの「一定距離」という概念の理解
- 視覚的な直感の形成

**実践的活動**:
- 黒板にコンパスで円を描く
- 直径を引く
- 円周上の点から直径に対する角度を測る
- 「その角度は常に 90 度のように見える」という観察

**AI による支援**（Jupyter AI）:
- 生徒が描いた円の性質について議論
- 「なぜそう見えるのか？」への仮説形成

---

### 3️⃣ **01_geogebra.ipynb** — GeoGebra での操作的理解

**学習内容**:
- GeoGebra の基本操作（点、線、円の作図）
- 動的な操作による理解
- 点をドラッグして「性質の変わらなさ」を観察

**操作的活動**:
- 単位円 $x^2 + y^2 = 1$ を定義
- 中心 $O = (0, 0)$ を配置
- 直径上の点 $C$ をドラッグ
- 円周上の点 $P$ を配置
- 角度 $\angle OPC$ を測定 → 常に 90 度であることを実験的に確認

**GeoGebra での作図**:
```
1. Circle eq1: x² + y² = 1
2. Point O = (0, 0)
3. Point P = (1, 0)
4. Line f: Line[O, P]
5. Point C = Point[f]  (直径上の自由な点)
6. Line g: PerpendicularLine[C, f]
7. Point D = Intersect[eq1, g]  (円との交点)
8. Angle angle_OPC: Angle[O, P, C]
```

---

### 4️⃣ **01_protocol.ipynb** — Construction Protocol

**学習内容**:
- GeoGebra の「Construction Protocol」（作図手順書）の概念
- 幾何学的な「層構造」（Level 0 → 4）
- 依存関係のグラフ化

**Construction Protocol テーブル**:

| # | Name | Type | Definition | Value | Layer |
|----|------|------|-----------|-------|-------|
| 1 | eq1 | Circle | $x^2 + y^2 = 1$ | Equation | 0 |
| 2 | O | Point | Center[eq1] | (0, 0) | 0 |
| 3 | P | Point | Point[eq1] | (1, 0) | 0 |
| 4 | f | Line | Line[O, P] | $y = 0$ | 1 |
| 5 | C | Point | Point[f] | (0.5, 0) | 1 |
| 6 | g | Line | PerpendicularLine[C, f] | ... | 2 |
| 7 | D | Point | Intersect[eq1, g] | (0.5, 0.87) | 2 |
| 8 | E | Point | Intersect[eq1, g] | (0.5, -0.87) | 2 |
| 9 | angle | Angle | Angle[O, D, E] | 90° | 3 |
| 10 | check | Boolean | AreEqual[angle, 90°] | true | 4 |

**層構造の理解**:
- **Layer 0**: Free objects（ユーザーが設定）→ Circle, Center, Point on circle
- **Layer 1**: 基本的な派生（直線、基準点）
- **Layer 2**: 交点、副次的な構築
- **Layer 3**: 測定（角度、距離）
- **Layer 4**: 検証（ブール値、等式チェック）

**Python での依存グラフ可視化**:
```python
from ggblab_extra import ggb_parser
import networkx as nx
import matplotlib.pyplot as plt

parser = ggb_parser()
# (dataframe を populate)
parser.parse()

# グラフの可視化
pos = nx.spring_layout(parser.G, seed=42)
nx.draw(parser.G, pos, with_labels=True, node_color='lightblue', arrows=True)
plt.title("Thales' Theorem: Dependency Graph")
plt.show()
```

---

### 5️⃣ **01_verify.ipynb** — ggblab による検証

**学習内容**:
- ggblab の `SceneVerifier` を使った自動検証
- 各オブジェクトの有効性確認
- 層ごとの検証結果の解釈

**検証コード**:
```python
from ggblab import GeoGebra, ggb_file
from ggblab_extra import ggb_parser
from ggblab_extra.scene_verification import SceneVerifier
import polars as pl

# 初期化
ggb = await GeoGebra().init()

# Thales 構築をロード（保存済みの .ggb ファイル）
c = ggb_construction()
c.load("chapters/01/scenes/thales_scene.ggb")
await ggb.function("evalXML", [c.geogebra_xml])

# Construction Protocol をデータフレームに変換
construction = {}
for obj in await ggb.function("getAllObjectNames"):
    info = await ggb.function(
        ["getObjectType", "getCommandString", "getValueString", "getCaption", "getLayer"],
        [obj]
    )
    construction[obj] = info

df = pl.DataFrame({
    'Name': list(construction.keys()),
    'Type': [v[0] for v in construction.values()],
    'Command': [v[1] for v in construction.values()],
    'Value': [v[2] for v in construction.values()],
    'Caption': [v[3] for v in construction.values()],
    'Layer': [v[4] for v in construction.values()],
}, strict=False)

# 依存グラフを解析
parser = ggb_parser()
parser.initialize_dataframe(df=df)
parser.parse()

# 自動検証
verifier = SceneVerifier(ggb, parser)
results = await verifier.verify_all()
print(verifier.summary(results))

# 期待される出力：
# Verification Summary
# ==================================================
# Total objects: 10
# Passed: 10 ✓
# Failed: 0 ✗
# Pass rate: 100.0%
```

**層ごとのチェック**:
- Layer 0 → 「基本的な円と点が定義されているか？」
- Layer 1 → 「直線と基準点が正しく導出されているか？」
- Layer 2 → 「交点が正しく計算されているか？」
- Layer 3 → 「角度が 90 度に近いか？」(許容誤差 < 1e-6)
- Layer 4 → 「Thales の定理が真か？」(Boolean チェック)

---

### 6️⃣ **01_theorem.ipynb** — Thales の定理の導出と証明

**学習内容**:
- Thales の定理の数学的証明
- 複数の証明方法（幾何学的、代数的、ベクトル的）
- 定理の一般化と拡張

#### **証明 1: 幾何学的証明**

円の中心を $O$、直径の端点を $A$, $B$、円周上の任意の点を $P$ とする。

1. $|OA| = |OB| = |OP| = r$ (半径)
2. △OAP は二等辺三角形 → $\angle OAP = \angle OPA$ (= α)
3. △OBP は二等辺三角形 → $\angle OBP = \angle OPB$ (= β)
4. △ABP の内角の和: $\angle PAB + \angle ABP + \angle APB = 180°$
5. $α + β + (α + β) = 180°$
6. $2(α + β) = 180°$ → $α + β = 90°$ = $\angle APB$

#### **証明 2: 代数的証明（Python）**

```python
import numpy as np

# 単位円上の点
O = np.array([0, 0])
A = np.array([1, 0])
B = np.array([-1, 0])
P = np.array([np.cos(theta), np.sin(theta)])  # theta ∈ (0, π)

# ベクトル
PA = A - P
PB = B - P

# 内積
dot_product = np.dot(PA, PB)

# 垂直ならば内積 = 0
assert abs(dot_product) < 1e-10, f"Not perpendicular: {dot_product}"
print(f"✓ Thales' theorem verified: PA · PB = {dot_product}")
```

#### **拡張: 逆定理**

> **Thales の逆定理**: 直径の上にある点から見た円周角が 90 度なら、その点は円周上にある。

この逆定理も ggblab で検証可能：
```python
# 角度が 90 度の点が、常に円周上にあるか？
for theta in np.linspace(0, 2*np.pi, 360):
    P = np.array([np.cos(theta), np.sin(theta)])
    # (検証コード)
    assert np.linalg.norm(P) == 1.0  # 円周上
```

---

## 学習成果の評価

### 知識的評価
- Thales の定理を説明できるか？
- なぜそれが真なのか、複数の方法で証明できるか？

### スキル的評価
- GeoGebra で自由に作図できるか？
- ggblab を使ってコンピュータで検証できるか？
- Python で数値計算・グラフ化できるか？

### 概念的評価
- 幾何学の依存構造が、プログラミングのスコープに対応していることを理解したか？
- ビジュアルと記号の対応関係を認識しているか？

---

## 15 週間カリキュラムの全体構成

textbook-2026 は、大学初年次向け少人数ゼミナールの **15 週間（約 4 ヶ月）の完全カリキュラム**です。以下の段階的構成により、古代ギリシア幾何学から現代計算科学への連続性を体験的に理解します：

### **第 1 段階：基礎と直感（Week 1-2, Chapter 01-02）**
- **Week 1 — Chapter 01（本章）**: Thales の定理の再創造
  - 古代幾何学の転機：直径（diameter）の本質
  - GeoGebra と Python による双対的対話
  - 視覚的直感から論理的形式化への上昇

- **Week 2 — Chapter 02**: 円の層構造
  - 円の「見方」の多様性（幾何学的、代数的、パラメトリック）
  - Construction Protocol による依存関係の明示

### **第 2 段階：発展と拡張（Week 3-7, Chapter 03-07）**
- **Week 3-4 — Chapter 03-04**: 公式から図形へ
  - 代数表現と幾何図形の対応関係
  - 幾何平均と相似の定理

- **Week 5-6 — Chapter 05-06**: 楕円と円錐曲線
  - 楕円の焦点と距離和の不変性
  - 円錐截断としての楕円、放物線、双曲線
  - **Dandelin の球による再構築の本質**（二つの円の接線性）

- **Week 7 — Chapter 07**: 接線と共通接線
  - 円への接線（Tangent lines）
  - 二つの円の外的・内的共通接線
  - 楕円の焦点性への隠された対称性

### **第 3 段階：科学的飛躍（Week 8-10, Chapter 08-10）**
- **Week 8 — Chapter 08**: 観測者の変化（Tycho Brahe → Kepler）
  - 天体観測の精密化
  - 円形軌道から楕円軌道への転換

- **Week 9 — Chapter 09**: Kepler の法則
  - 楕円軌道における焦点の動的意義
  - 物理学への適用による幾何学の再評価

- **Week 10 — Chapter 10**: Apollonius から Kepler へ
  - Apollonius の直径論の限界
  - Kepler による楕円焦点の発見
  - 古典幾何学と物理科学の統合

### **第 4 段階：古典の再評価（Week 11-13, Chapter 11-13）**
- **Week 11 — Chapter 11**: Archimedes の方法論
  - 穎尽法（Method of Exhaustion）
  - π の計算と極限の概念

- **Week 12 — Chapter 12**: Descartes と Newton
  - 座標系による幾何学の代数化
  - 微分・積分の萌芽

- **Week 13 — Chapter 13**: 投影と変換
  - 射影幾何学の基礎
  - 円錐曲線の統一的理解

### **第 5 段階：現代への連続性（Week 14-15, Chapter 14-15）**
- **Week 14 — Chapter 14**: 曲率と微分幾何
  - 局所的な「曲がり具合」の定量化
  - Archimedes から微分幾何学へ

- **Week 15 — Chapter 15**: 球面上の幾何学
  - 非ユークリッド幾何学への入り口
  - 古典から現代数学への展望

### **このカリキュラムの学習パス**

```
Week 1: なぜ円か？ → Thales の定理の再創造
        ↓
Week 2-4: 円とは何か？ → 層構造、相似、公式と図形
        ↓
Week 5-7: 円を超える → 楕円、円錐曲線、接線（Dandelin の再構築）
        ↓
Week 8-10: 天体物理への飛躍 → Kepler、物理現象による幾何学の検証
        ↓
Week 11-13: 古典の再評価 → Archimedes, Descartes, Newton の方法論
        ↓
Week 14-15: 現代への展開 → 微分幾何、非ユークリッド幾何学
        ↓
学習成果: 古代幾何学から現代計算科学への連続性を体験的に理解
```

---

## 次のステップ

**Chapter 02（Week 2）** では、Thales の定理で学んだ「層構造」を、より一般的な円の性質へ拡張します：

- 円の「見方」の多様性：幾何学的 vs 代数的 vs パラメトリック
- Construction Protocol による依存関係グラフの詳細分析
- 複数の円の相互関係（交点、接線、相似）
- ggblab による自動検証の実装パターン

---

## 本カリキュラムが達成する体験的学習

1. **Week 1-7**: 直感から形式化へ
   - Thales の定理を起点に、古代幾何学の思考体系を再現
   - Dandelin の球による「隠された対称性」の発見

2. **Week 8-10**: 現象から真理へ
   - 天体観測がいかに幾何学の理解を変えたかを体験
   - Kepler の楕円焦点の発見の必然性を理解

3. **Week 11-15**: 古典から現代へ
   - Archimedes から Newton、そして微分幾何学への連続性
   - 形式化の進化が、問題解決の精密性をもたらすことを認識

---


