---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.2
kernelspec:
  name: julia-1.12
  language: julia
  display_name: Julia 1.12
---

# 第5回: 重心を Archimedes の天秤で正当化する — アフィンの細片と DataFrame 外部化

**主題**: 重心 (Centroid) を **Archimedes の天秤** を軸に導き、その議論が **正当** であることを示す。続いて **バリセントリック座標が何故許容されるか** を、同じ天秤の言い換えとして解説する。

**副題**: 天秤の細片は三角形の軸 (中線) に直交しない。それでも正しいのは、重心が **アフィン (斜行で不変) な概念** だからである。直交は要らない。

**学習目標**

1. 重心を **Archimedes の天秤（細片を中線に沿って釣り合わせる）** で導き、それが直交を仮定せず **アフィンに正当** であることを理解する。
2. **バリセントリック座標（頂点に等質量）が何故許容されるか** を、天秤の離散版・三角形固有の「板の重心（積分）= 頂点和」定理として説明できる。
3. 作図履歴を Construction Protocol → DataFrame として **外部化** し、同じ点への到達を座標値でなくデータ構造で検査できるようになる。
4. **(伏線)** アフィンの自由は Apollonius の発見であり、重心ではそれが **接線を含まないから clean** である。接線をアフィン座標で扱うときに二千年の封印が起こる —— それは後の回（円錐切断）で回収する。

+++

:::{note} 本書の参照記法 —— `MCP lancedb-rag「…」project=…` を @Codex にどう尋ねるか
:class: note
本書と関連教材の参照は、ファイルへの直リンではなく **MCP 問い合わせ記法** で書かれている:

> `MCP lancedb-rag「<クエリ>」project=<プロジェクト>`

これを見たら、@Codex に次のように入力して中身を引く（ファイルのパスを辿る必要はない）。たとえば本文中の `MCP lancedb-rag「教材オーサリング規約 ggblab セル シンボル」project=textbook` を読みたいとき:

````text
@Codex lancedb-rag で「教材オーサリング規約 ggblab セル シンボル」を project=textbook で調べて、要点を教えて。
````

@Codex は教材ドメイン RAG（lancedb-rag）を検索し、該当文書（この例なら著者向けの作図セル規約）を返す。`project` は探す範囲の指定で、`textbook`=本教材群 / `conversations`=設計対話 など。リンク切れの心配がなく、「検索して辿り着く」 という実際のワークフローのまま参照できる。
:::

+++

## 0. 前回 (第 4 回) の振り返り

前回は **外心** を Thales の二定理だけから組み立てました：

- 公準 (1) 円は直径によって二等分される
- 公準 (2) 二等辺三角形の底角は等しい
- → Lemma 1: 弦の垂直二等分線は中心を通る
- → 主命題: 三辺の垂直二等分線は一点 O で交わる（外心）

**前提一つ、命題連鎖の最短経路** という構造でした。

今回は逆に、**前提を複数並べて、同じ結論に至る** という構造を扱います。

```{tip} 先週仕込んだ語彙が今日 instance 化する
:class: tip

L4 §0 で「**嫌い → 抽象化**」 という閃きの pattern を articulate した (Babylonia/Napier/Leibniz/Anthropic 系譜)。今日は、Möbius がバリセントリック座標 (1827) で literal にこの pattern を経た事例に出会う (§4.2)。

L4 §0 のファミマ入店音 reframe (同じ scope を異なる文脈で実装) も今日体験する: 「重心」 という同じ点を、天秤/面積/座標という三つの異なる前提で arrange する (§4)。**連続講義の体験は、先週の語彙が今週使える** ようになることに価値の一部がある。
```

+++

:::{admonition} 目標の確認 —— @Codex で達成目標を引く（L5 から導入）
:class: note
今回から、達成目標を **@Codex に lancedb-rag（教材ドメイン RAG）で確認**してもらえる。皆さんは Jupyter AI の @Codex でこの教材コーパスを検索できる。
（プロンプト例）@Codex この回（L5 重心）の達成目標を lancedb-rag で調べて、要点を整理して。
自分の理解と照らし、特に「天秤が何故正当か」「バリセントリックが何故許容されるか」 にズレや腑に落ちなさがあれば書き留めてから先へ進む。
:::

+++

## 1. 三つの前提 — 重心への三つの道

重心は次の三つの前提のどれからでも導けます：

| 前提 | 起源 | 性質 |
|---|---|---|
| **Archimedes の天秤** | 紀元前 3 世紀、Archimedes 『On the Equilibrium of Planes』 | 物理的直観：質点の平衡で重心を定義 |
| **面積分割** | Euclid 流、純粋な幾何 | 中線は三角形を等面積に分ける |
| **バリセントリック座標** | 1827 年、Möbius 『Der barycentrische Calcül』 | 線形結合の係数で点を表す。座標選択に依存しない |

**問い**: 三つの異なる前提が **同じ点** に至るのは偶然か？  
あるいは、何かが保証しているのか？

参考: [Centroid - Wikipedia](https://en.wikipedia.org/wiki/Centroid)、[Barycentric coordinate system - Wikipedia](https://en.wikipedia.org/wiki/Barycentric_coordinate_system)、[August Ferdinand Möbius - Wikipedia](https://en.wikipedia.org/wiki/August_Ferdinand_M%C3%B6bius)

+++

## 2. ggblab セットアップと三角形の準備

```{code-cell}
using Pkg
Pkg.activate("../..")
Pkg.resolve()
Pkg.instantiate()
using GeoGebra
ENV["GGB_DIRECT_TRANSPORT"] = "true"
```

```{code-cell}
inject_applet()
```

```{code-cell}
@ggb :const :new
@ggb A=(2, 1)
@ggb B=(-1, 2)
@ggb C=(0, -1)
@ggb Polygon(:A, :B, :C)
```

```{tip} @Codex 多層対話 model (L4 §2 で詳述)
:class: tip

L4 §2 で説明した三層対話 model (context / tools / post-processing) は、L5 でも同じく機能する。今日 ggblab セットアップしたアプレットは @Codex にとっての「環境 (tools)」 になる。

「重心がなぜ一点に集まるか」 のような **答えは知っているが理由が混みあった** 問いを @Codex に聞くと、Möbius バリセントリック座標を出してくる可能性が高い。これは LLM の standard answer で、L5 §6 で見るように **答えと「なぜ動くか」 のギャップ** を体感する素材になる (Bender 「Stochastic Parrot」)。
```

## 3. 解くべき命題を発見する

中線（各頂点から対辺の中点への線分）を三本引き、観察します。

```{code-cell}
@ggb M_a = Midpoint(:B, :C)
@ggb M_b = Midpoint(:C, :A)
@ggb M_c = Midpoint(:A, :B)
@ggb s_a = Segment(:A, :M_a)   # 中線(規律1: 各オブジェクトを一意のシンボルに)
@ggb s_b = Segment(:B, :M_b)
@ggb s_c = Segment(:C, :M_c)
```

**観察**: 三本の中線が一点で交わる。さらに、その点は中線を **2:1 に内分** しているように見える。

**問い**: B, C の取り方を変えても成り立つか？ 成り立つとしたら、なぜか？

前回と同じ構造の問い — 観察から命題への移行です。

+++

## 4. Archimedes の天秤を軸に — 正当性、そしてバリセントリックは何故許容されるか

重心への道は複数あるが、本回は **Archimedes の天秤を軸** に据え、それが正当であることを示す。続いて **バリセントリック（頂点に等質量）が何故許容されるか** を、天秤の言い換えとして解説し（§4.2）、最後に面積分割が独立な事実として同じ点に来ることを確認する（§4.3）。

+++

### 4.1 Archimedes の天秤 — 細片を中線で釣り合わせる（主筋）

Archimedes 自身の議論（『平面の釣り合い』）は、**頂点に質点を 3 つ** 置くのではなく、**三角形を底辺 BC に平行な細片に分け、天秤で釣り合わせる** ものだった。

1. BC に平行な細片を考える。各細片は一様な棒で、その重心は **中点**。
2. その中点は、A から $M_a$（BC の中点）への **中線上に乗る**（下の作図で確認）。
3. ゆえに全細片の重心は中線上 → **三角形の重心は中線 $AM_a$ 上**。
4. 中線に沿って細片の幅は線形に増える（相似）。中線方向の距離を $t\in[0,1]$ とすると質量 $\propto t$、重心位置は $\int_0^1 t\cdot t\,dt \big/ \int_0^1 t\,dt = (1/3)/(1/2) = 2/3$ → **AG:GM_a = 2:1**。
5. 同じ議論を 3 辺で行うと、三中線すべてに同じ点が乗る → **中線は一点で交わる**（concurrence も同時に出る）。

```{important} 正当性の核 — 直交は要らない（アフィン）
:class: important
細片は一般の三角形では **中線に直交しない**（斜行）。それでも手順 2「細片の中点が中線上に乗る」が成り立つのは、**中線が BC に平行な弦を二等分する** という **アフィン事実**（剪断で不変、直交を使わない）だからである。二等辺三角形（軸 ⊥ 底辺）は特殊例にすぎない。

手順 4 の極限（細片 → 0）は積尽法 (exhaustion) で、Archimedes が **二重背理法で厳密化** した（放物線求積の $4/3 = 1 + 1/4 + 1/16 + \cdots$ と同じ厳密さ）。だから天秤は「物理的直観」 にとどまらず **論理的に正当** である。
```

**GeoGebra で「細片の中点が中線上」を確認する**（直交を一切使っていないことに注意）:

```{code-cell}
@ggb t = Slider(0, 1)
@ggb D = "A + t*(B - A)"        # 辺 AB 上の点
@ggb E = "A + t*(C - A)"        # 辺 AC 上の点
@ggb s_DE = Segment(:D, :E)   # BC に平行な細片(弦)
@ggb N = Midpoint(:D, :E)     # 細片の中点
@ggb m_a = Line(:A, :M_a)     # A からの中線
```

スライダ $t$ を動かすと、細片 $s_{DE}$ は BC に平行に動き、その中点 $N$ は **常に中線 $m_a$ 上** に乗る（$N = A + t\,(M_a - A)$ ゆえ）。中線は BC に直交していないのに、である。これが、天秤が斜行（アフィン）でも正当である理由の可視化。

> **3 点質点版との関係**: 「頂点に等質量 3 つ → 重心 $(A+B+C)/3$」 は、この細片積分の **三角形に固有の縮約** である。なぜ縮約が許されるか（積分 = 頂点和）は §4.2 で示す。

```{note} 等積変換（受験で習うアレ）は、せん断 = アフィン
:class: note
「等積変形」（三角形の頂点を底辺に平行に滑らせても面積が変わらない）は **せん断（shear）= 行列式 1 のアフィン変換**。重心が「斜行で不変」 とは、まさにこの等積変換で動かないこと。ただしアフィンは **面積（比）は保つが、長さ・角・垂直は保たない**。Euclid のピタゴラスの定理（命題 I.47、長さの² = 計量命題）も、実は **等積変換（同じ底辺・同じ平行線間の三角形は等積、I.37/41）+ 回転（合同）** で、計量の内容を面積に encode して操作している。Archimedes の放物線求積（面積比 4/3）も同じくアフィン量。→「せん断で何が保たれ（面積）何が崩れるか（長さ・角）」 が、重心（L5 アフィン）と内心（L6 計量）を分ける軸であり、円錐曲線（L11+）まで続く。
```

**せん断を行列で書いて確かめる。** せん断 $S=\begin{pmatrix}1&k\\0&1\end{pmatrix}$ は $\det S = 1$ ゆえ面積を保つ（等積変換）。これを三角形に適用し、**重心がせん断と可換**（＝アフィン共変）であることを見る。

```{code-cell}
# 等積変換 = せん断 S。横せん断は det = 1 ゆえ面積を保つ。
@ggb k = Slider(-2, 2)
@ggb S = {{1, k}, {0, 1}}
@ggb detS = Determinant(:S)         # = 1（等積 = 面積保存）
```

```{code-cell}
# 三角形 ABC にせん断を適用（ApplyMatrix）。形は歪むが面積は不変。
@ggb A_s = ApplyMatrix(:S, :A)
@ggb B_s = ApplyMatrix(:S, :B)
@ggb C_s = ApplyMatrix(:S, :C)
@ggb tri_s = Polygon(:A_s, :B_s, :C_s)
```

```{code-cell}
# 重心はせん断と「可換」: せん断後の重心 = 元の重心をせん断したもの。
@ggb G     = "(A + B + C) / 3"         # 元の重心
@ggb G_{s}   = "(A_s + B_s + C_s) / 3"   # せん断後の三角形の重心
@ggb G_{byS} = ApplyMatrix(:S, :G)     # 元の重心を同じせん断で移した点
```

スライダ $k$ を動かすと三角形は等積に歪むが、$G_s$（歪んだ三角形の重心）と $G_{byS}$（元の重心を同じせん断で移した点）は **常に一致** する。これが「重心はアフィン共変」 の意味 —— せん断の前に重心を取っても後に取っても同じ。中線は中線に、$2{:}1$ は $2{:}1$ に写る。$\det S = 1$ ゆえ面積も不変（等積）。

> この同じ `ApplyMatrix` を、後の円錐曲線（L11+）では **接線・斜行直径** に適用する（`geogebra/diameter/` の構成がそれ）。そこでは保つ量（面積比）と崩れる量（接線の計量的役割）の境界が問題になる —— 本日のせん断は、その道具の **clean な先行使用** である。

+++

### 4.2 バリセントリック座標による証明

点 P を **バリセントリック座標** で次のように表す：
$$ P = \alpha A + \beta B + \gamma C, \quad \alpha + \beta + \gamma = 1 $$

1. 辺 BC の中点 $M_a$ は $M_a = (0, 1/2, 1/2)$。
2. A から $M_a$ への中線上の点はパラメータ $t \in [0, 1]$ で
    $$ P(t) = ((1-t), t/2, t/2) $$
    （係数和は $1$ を保つ）。
3. 三本の中線が一点で交わるなら、対称性より $\alpha = \beta = \gamma$ が成立。
4. $\alpha + \beta + \gamma = 1$ より $\alpha = \beta = \gamma = 1/3$。
5. すなわち **重心 $G = (1/3, 1/3, 1/3)$**。
6. $P(t) = G$ となる $t$ は $t = 2/3$。これは AG が中線 $AM_a$ の $2/3$、すなわち **AG:GM_a = 2:1**。

**何故バリセントリックが許容されるか**（3 点質点 $(A+B+C)/3$ が「簡素化しすぎ」ではない理由）:

1. **アフィン不変** — 座標系の取り方に依存しない。これは Apollonius が円錐曲線で発見した「どの直径を選んでもよい」 = アフィンの自由（discussion_memo §4（MCP `lancedb-rag`「Apollonius 通径 接線封印 斜交直径」project=conversations））と同じ構造。重心はアフィン概念なので、斜行でも何も失わない。
2. **三角形では「板の重心（積分）= 頂点和（離散）」が恒等的に成り立つ**（これは定理）。一般三角形で記号積分すると $\displaystyle \frac{1}{\text{Area}}\iint_{\triangle}\mathbf{r}\,dA = \frac{A+B+C}{3}$（ヤコビアンが分子分母で打ち消す）。だから天秤の細片積分（§4.1）を **3 点和に縮約しても exact**。
3. **ただし三角形固有** — 四角形では頂点和 ≠ 板の重心（割れる）。バリセントリックが「重心」を与えるのは三角形（単体）の特権。

→ すなわち **天秤（細片積分・§4.1）とバリセントリック（頂点和・本節）は独立な二証明ではなく、同じ内容を連続（積分）と離散（和）で書いたもの**。三角形でだけ両者が一致する。Möbius (1827) の発想は、座標選択の自由を構造的に吸収する画期である（次の callout）。

```{important} 「嫌い → 抽象化」 pattern が今 literal に起こっている
:class: important

Möbius (1827) は **座標選択の煩わしさを嫌った**。Descartes の (x, y) 座標は便利だが、原点・軸の取り方によって同じ点が違う数で表される。「重心」 のような **配置に内在する量** を扱うのに、外部的な座標選択を経由するのは不自然である。

この嫌悪が **バリセントリック座標** $P = αA + βB + γC$ という抽象化を生んだ。係数 $(α, β, γ)$ は、原点や軸の選び方に依存しない。三角形の頂点 $A, B, C$ 自身を「内部の参照点」 として使う。

これは L4 §0 で仕込んだ「**嫌い → 抽象化**」 という閃きの pattern の literal instance である:

- Babylonia: 大きな数の掛け算が嫌い → log-log table の前駆
- Napier (1614): 大きな数の掛け算が嫌い → 対数表
- Leibniz: Newton の流率表記が嫌い → 現代の $dx, dy$ 記法
- **Möbius (1827)**: 座標選択が嫌い → バリセントリック座標
- Anthropic: マルチターン RL の不安定が嫌い → Constitutional AI

「嫌い」 を articulate できる人だけが抽象化の入口に立つ。
```

+++

### 4.3 三つの道の関係 — 連続（天秤＝面積）と 離散（バリセントリック）

「面積分割」 という第三の道をよく聞くが、それは §4.1 の天秤と **別物ではない**。中線が三角形を等面積に二分する事実（$\triangle ABM_a = \triangle ACM_a$、底辺が等しく高さが共通）は、細片積分（§4.1）を面積の言葉で言い換えたもの —— どちらも **連続体（板）の重心** を見ている。

整理すると、本質的に道は二筋:

| | 連続（積分） | 離散（和） |
|---|---|---|
| 何 | 天秤の細片積分 ＝ 面積（板）の重心（§4.1） | 頂点に等質量 ＝ バリセントリック（§4.2） |
| 式 | $\dfrac{1}{\text{Area}}\displaystyle\iint_{\triangle}\mathbf{r}\,dA$ | $\dfrac{A+B+C}{3}$ |

そして **三角形では両者が恒等的に等しい**（§4.2-2 の定理）。四角形では割れる。

→ 「三つの前提が同じ点に来る」 は正しいが、**独立な三証明ではない**。連続（天秤＝面積）と離散（バリセントリック）の **二筋** が、三角形固有の定理で一致しているのである。AG:GM_a = 2:1 も両筋で一致。

では、同じ点を指していることを、目視でなく **データ** でどう検査するか？ → §5。

+++

:::{tip} 進む前に — 自分が気持ち悪いと感じる箇所を書き留める
:class: tip
§4.1(天秤)/ §4.2(バリセントリック)/ §4.3(連続-離散整合)を読んで、
**どこに「気持ち悪さ」を感じたか**(または感じなかったか)を書き留めてから、
次の attention callout へ進んでください。先生自身の articulation を読む前に、
自分の感覚を持つことが重要です。
:::

+++

:::{attention} 正直な隙はどこにあるか —— 重心は clean、開いているのは「接線」
:class: attention

自然な感覚として、三証明のどれかに「気持ち悪さ」を感じた読者もいるかもしれない。結論を先に述べると、その気持ち悪さは重心の議論にはなく、後で扱う接線の議論にある。
今年度は、議論の末にそれを **解消** して据え直す:

- **天秤（§4.1）は正当**。「物理直観だから曖昧」 ではない。細片の中点が中線に乗るのは **アフィン事実**（直交不要）、極限は **積尽法で厳密**。隠れていた前提（細片を一つの重心に合成してよい、という合成原理）を明示すれば、直観的かつ論理的が両立する。
- **バリセントリック（§4.2）が許容されるのもアフィンゆえ**。3 点和は細片積分の三角形固有の縮約で、定理として exact。
- だから **重心の議論に「ごまかし」 は無い**（純アフィン。接線も、極限の言語化不在も登場しない）。

**本当に開いている隙は、重心ではなく「接線」 にある。** Euclid 第 III 巻命題 16 は、接線の局所構造を量として正面から扱うことを避けた —— 原論で唯一、量の比較可能性が公理系から逸脱する箇所（discussion_memo §4（MCP `lancedb-rag`「Apollonius 通径 接線封印 斜交直径」project=conversations））。Apollonius はこの **定義されない接線** の上に通径を築いて二千年封印した。気持ち悪さを感じるべきはそこであって、重心ではない。次の callout で、その伏線を張る。
:::

+++

```{note} 伏線 —— アフィンの自由は Apollonius の発見。clean に使うか、接線を封印するか
:class: note

今日使った「斜行でも中線が平行弦を二等分する」 ＝ **アフィンの自由**（どの直径・どの軸を選んでもよい）は、実は **Apollonius が円錐曲線で発見したもの**（discussion_memo §4（MCP `lancedb-rag`「Apollonius 通径 接線封印 斜交直径」project=conversations））。Menaechmus の特権的な直交軸を捨て、斜交直径系で円錐曲線を統一した発見である。

重心では、このアフィンの自由を **大域量（面積・モーメント）にだけ** 使うので clean —— 接線も極限も要らない。
だが Apollonius は同じアフィンの自由を **接線（局所構造）の記述** に使い、通径で接線を静的に封じた。これが二千年の停滞の根（Higashida の診断、未だ反駁されていない）。封印は Descartes（法線）・Fermat（極限）・Newton（運用）を経て Cauchy/Weierstrass で解かれ、現代では **判別式 $D=0$ / SymPy で「重解 ＝ 接線」 を言語化** して回収する。

→ **重心（L5）＝ アフィンを大域に clean に使う先行例**。**接線の封印とその回収（円錐切断・L11+）＝ 同じアフィンを局所に使うときの二千年の物語**。今日はその伏線を踏んだ。
```

+++

## 5. ggblab_extra 初出 — Construction Protocol を DataFrame として外部化する

ここで、本講義シリーズの **道具のロードマップ** の第一段階に進みます。

+++

### 5.1 なぜ今これが必要か

三つの起点から「同じ点」に到達したことを保証したい。しかし、目で見ただけでは、誤差で重なって見えているだけかもしれません。

GeoGebra アプレットの **内部** には、作図された各オブジェクト（点・線・円など）の名前・型・定義式・座標値・依存元がすべて記録されています。これを **Construction Protocol** と呼びます。

Construction Protocol を Python/Julia の **DataFrame** として取り出せば、三つの起点で得られた重心の座標を **数値として直接比較** できます。

→ 「アプレットの内部状態を外部化する」という操作は、本講義のここから先で **何度も使う基本作法** になります。

+++

### 5.2 何が起きているか — 最小デモ

現在のアプレットの作図状態を DataFrame として取り出します。

（ggblab_extra の Python 側機能を呼び出す形になります。Julia の `juliacall` 経由、または Python セルで実行します。先生の環境での正確な呼び出し方法に合わせて調整してください。）

```{code-cell}
# Archimedes の天秤による重心の構成（GeoGebra 組み込みコマンドで）
@ggb G_{balance} = TriangleCenter(:A, :B, :C, 2)  # 2 = Centroid
```

```{code-cell}
# バリセントリック座標による重心
@ggb G = "(A + B + C) / 3"
```

```{code-cell}
# 中線の交点による重心（§3 で定義した中線 s_a, s_b を使う — 規律1: Intersect を入れ子にしない）
@ggb G_{medians} = Intersect(:s_a, :s_b)
```

### 5.3 DataFrame として取り出して比較する

三つの構成 $G_{\text{balance}}$, $G_{\text{bary}}$, $G_{\text{medians}}$ を DataFrame に並べて、座標値を比較します。

```{code-cell}
# Construction Protocol を DataFrame として外部化
# Python 側の ggblab_extra を呼び出す（環境に応じて API を調整してください）
#
# 例: textbook-2025 examples_by_dual_coding/eg00.ipynb で使用された形：
#   df = await ConstructionIO.initialize_dataframe(ggb, use_applet=True)
#   df[["Name", "Type", "Command", "Value"]]
#
# Julia 側からは juliacall 経由、または PyCall でアクセスします。

using PythonCall
pl = pyimport("polars")
ggb = pyimport("ggblab")
ggblab_extra = pyimport("ggblab_extra")
ggb.file = pyimport("ggblab.file").ggb_file()
ggb.parser = pyimport("ggblab.parser").ggb_parser()
ggb.schema = pyimport("ggblab.schema").ggb_schema()
ConstructionIO = ggblab_extra.ConstructionIO
```

```{code-cell}
df = @await ConstructionIO.initialize_dataframe(ggb, use_applet=true)
df_centroids = df.filter(pl.col("Name").is_in(["G_{balance}", "G", "G_{medians}"]))
df_centroids
```

**DataFrame で見ると、何が起きるか**：

- 三行のうち、`Value` 列の座標が（数値誤差の範囲で）一致している → 三つの起点が同じ点を指していることが **データレベルで確認** できる
- `Command` 列を見ると、三つが **異なる手続き** で構成されていることが分かる → 異なる前提から同じ結論に至った経路が記録されている
- `Type` 列は全て `point` → 三つとも点として扱われている

観察を **目視ではなくデータで検証** できる。これは Apollonius 期の幾何学が手作業で行っていた作業を、現代のツールで一段階上にした行為です。

+++

:::{admonition} 進捗の確認 —— セル出力を @Codex に読ませる
:class: tip
ここまでで「三つの構成が同じ点か」 を DataFrame で検査した。@Codex は **jupyter-server-mcp であなたのセル出力を直接読む**（プロンプトだけの対話ではない）。
（プロンプト例）@Codex ここまでのセル出力を読んで、達成目標①（天秤の正当性）②（バリセントリックの許容理由）にどこまで到達したか、まだ埋まっていないセルはどこか、具体的に挙げて。
:::

+++

### 5.4 これが汎用的にどこに効くか — 将来の武器

「アプレットの内部状態を外部 DataFrame に取り出す」という操作は、**他のあらゆる場面で再演されます**：

| 分野 | 同じ操作 |
|---|---|
| データサイエンス | データベースから pandas/polars にロードする |
| Web API 開発 | レスポンス JSON を構造化データに変換する |
| 計測 / 実験 | センサーログを DataFrame にして解析する |
| 機械学習 | モデルの中間表現を可視化する (TensorBoard, MLflow) |
| 数値計算 | シミュレーションの内部状態を時系列 DataFrame として外部化 |

**共通する思想**：**「ブラックボックスの中身を、検査可能なデータ構造に変換する」**。これは現代エンジニアリングの最も基本的な作法の一つです。

Apollonius 期に隠蔽されていた接線の局所構造を、19 世紀の数学者たち（Cauchy/Weierstrass）が記号化と論理化で **外部化** したのと、同じ動きの現代版です。

+++

## 6. 停滞と画期の照射 — 物理直観の二千年

Archimedes の天秤による重心の議論は、**紀元前 3 世紀から二千年の間、物理的直観として受け入れられ続け** ました。「天秤を傾ける」という日常感覚で正しい答えに到達できる。

この議論自体は §4.1 の通り **当時すでに積尽法で厳密** だった（単なる物理的直観に留まらない）。待たねばならなかったのは厳密化ではなく、それを **座標非依存の一般的枠組みに抽象化** すること —— Möbius のバリセントリック座標 (1827) が座標選択を構造的に消した。

| 段階 | 何が画期だったか |
|---|---|
| 紀元前 3 世紀 (Archimedes) | 物理直観で重心を **正しく** 計算できた |
| 1637 (Descartes) | 座標を導入したが、重心の本質は座標の選び方に依存しないはず |
| 1827 (Möbius) | バリセントリック座標で **座標選択の自由を構造的に吸収** |
| 1844 (Grassmann) | 線形結合の代数を抽象化（ベクトル空間の先駆） |
| 19 世紀後半 (Gibbs/Heaviside) | ベクトル代数として物理に整備 |

→ **「個別に厳密な結果」と「座標非依存の一般的抽象」の間に二千年の遅延** があった（答えの正しさや証明の厳密さではなく、構造を明示し一般化する抽象の遅れ）。

現代の例：LLM が「動く答え」を出してくれるが、**「なぜそう動いているか」が明確でない** 状態にいる。Archimedes 期と Möbius 期の間の二千年と、現代の状況に構造的類似があります。

これは Bender et al. (2021) 「Stochastic Parrot」 が articulate した **form と meaning のギャップ** と同型である: LLM は適切な form (動く回答・正しい数値) を生成するが、生成手続きと意味の対応関係 (meaning) は内部に持たない。Archimedes 期の物理直観 (天秤を傾ける) は日常 form で正しい answer に到達したが、**なぜ正しいかの meaning** は Möbius (1827) を待たねばならなかった。LLM もこの二千年期と同型の段階にいる可能性がある。

+++

```{admonition} 今回の課題
:class: tip

**必修**
1. 三つの起点（天秤・面積・バリセントリック）のいずれか一つで重心 G を作図し、Construction Protocol を DataFrame として取り出して、AG:GM_a = 2:1 を数値で確認せよ。
2. 残り二つの起点でも同じ点に到達することを、DataFrame の `Value` 列で確認せよ。

**思考課題**
3. 「正しい答え」と「なぜ正しいかが明確な答え」の間に時間的ギャップがある事例を、身の回りから一つ挙げよ（前回課題の延長として）。例: 経験則として動いている技術、説明はないが効くと信じられている手法、etc.

*ヒント*: 第 15 回での立論の素材として記録を残してください。
```

+++

## 7. 今学期の実験 — 科学転生無双レポート (L5 version)

L4 で導入した「現代から歴史の数学世界に転生して、現代知識で無双する」 課題 (L4 §7) を、今日の重心テーマで継続する。

**課題** (L5 version):

> **1827 年の Möbius の時代に転生したと仮定し、「バリセントリック座標」 という抽象化を生まない代替戦略を一つ考案せよ**。なぜその代替戦略が当時の他の数学者に採用されず、Möbius の座標が prevail したかを、現代の知見から推測せよ。

または、より easy mode として:

> **紀元前 3 世紀の Archimedes の時代に転生したと仮定し、「天秤による重心の議論」 の代わりに、現代の知識で別の説明戦略を考案せよ**。Archimedes 自身がなぜ天秤 framework を選んだか、現代から推測せよ。

**評価軸** (L4 §7 と同じ):

1. 当時の知識・道具立てを尊重しているか (時代錯誤の有無)
2. 「無双できる場面」 と「無双できない場面」 を自己評価しているか (cf. L4 §7 で観察された AI の self-pivot 事例)
3. なぜ Möbius (または Archimedes) の元の戦略が prevail したかの構造的説明

**注記** (5/19 観察): L4 §7 で先生が試した instance では、@Codex (gpt-5.4-mini) が Turing 系に転生できないと self-pivot して Neumann を選んだ。L5 では、AI に「Möbius 期に転生して何を考案するか」 を聞いてみると、別種の self-assessment pattern が観察できる可能性がある。AI が「**ここは無双できない**」 と articulate する場面を意識的に捕まえること。

**提出**: L4 と同じ format (markdown または PDF、A4 1 枚程度)。第 15 回での立論の素材として記録を残す。

+++

:::{important} 授業末尾の自己評価 — `@Codex` に聞いてみる (任意、L4 から継続)

L4 で導入した **授業末尾の自己評価 prompt** を、L5 でも同じ template で試してください。**任意** です。

### copy-paste 用 prompt

````text
@Codex 今日のノートブックを全 cell 読んで評価してください。
次の三つを区別して articulate してください:

1. 自分で考えて書いた cell — 思考の痕跡が残っている部分
2. AI 委託で書いたが、理解して受け入れた cell — 動いて、なぜ動くか説明できる部分
3. AI 委託で書いたが、なぜ動くか説明できない cell — 動いているが、理解で未到達の部分

加えて: 今日の lesson 主題 (重心 / Archimedes 天秤 / Möbius バリセントリック /
DataFrame 外部化) に対する自分の到達度、完成しないまま残った問い、
次回 (L6 内心) への接続点。

特に L5 §4 「三つの証明、それぞれの納得できなさ」 callout について、
私自身がどこに「気持ち悪さ」 を感じたか、または感じなかったかを正直に articulate してください。

私の cell 配列を見て、誤魔化さず正直に articulate してください。
````

### L5 特有の観察ポイント

- **§4.2 Möbius 「嫌い → 抽象化」 callout** が L4 §0 の pattern instance として認識できているか。先週の語彙が今週使えているかの直接 signal
- **§4 attention callout** (三つの証明それぞれの納得できなさ) で、自分の proof skepticism が言語化できるか。先生自身が articulate した open problem に学生が向き合えるかの確認
- **§7 科学転生無双 (Möbius 期 / Archimedes 期)** で、`@Codex` 自身の自己評価 (無双できる/できない) を観察する課題と、本 prompt の **自己評価の対称性** を意識すると、AI-学生 二口採点の構造が立ち上がります

JupyterAI のやり取り log は LMS 経由で先生に届きます — 学期末立論 (第 15 回) の materials として、毎週の自己診断 log を蓄積。**「先週は気付かなかった AI 委託を今週は気付けるようになった」** が回を重ねるごとに起こるか、皆さん自身でも観察してみてください。
:::

+++

## 8. 次回への接続

次回 (第 6 回) は **内心 (Incenter)** に移ります。内心は外心とも重心とも違う系統 — **角の二等分線** の交点として定義されます。

今回扱った Construction Protocol → DataFrame は、第 6 回でも引き続き使います。**角の二等分線が一点で交わることをデータで検証** する場面です。

また、次々回 (第 7 回) では DataFrame をさらに一歩進めて **有向グラフ (networkx)** に変換します。Euler 線（外心・重心・垂心が一直線上に並ぶ）の理由を、座標計算ではなく **依存構造** から見ます。

道具のロードマップは ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook） を参照。

+++

## 参考文献

- [Centroid - Wikipedia](https://en.wikipedia.org/wiki/Centroid)
- [Barycentric coordinate system - Wikipedia](https://en.wikipedia.org/wiki/Barycentric_coordinate_system)
- [August Ferdinand Möbius - Wikipedia](https://en.wikipedia.org/wiki/August_Ferdinand_M%C3%B6bius)
- [Archimedes' On the Equilibrium of Planes (Heath 訳)](https://archive.org/details/worksofarchimede00archuoft)
- [Hermann Grassmann - Wikipedia](https://en.wikipedia.org/wiki/Hermann_Grassmann)
- [pandas](https://pandas.pydata.org/) / [Polars](https://pola.rs/) — DataFrame の現代的実装
- Bender, Gebru, McMillan-Major, Mitchell (2021) ["On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? 🦜"](https://dl.acm.org/doi/10.1145/3442188.3445922) FAccT — form と meaning の区別
- 前回授業 前回 第4回(外心)（MCP `lancedb-rag`「第4回 外心 Thales 嫌い 抽象化」project=textbook） — §0 「嫌い → 抽象化」 pattern、§7 科学転生無双レポート
- 道具のロードマップ ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook）
