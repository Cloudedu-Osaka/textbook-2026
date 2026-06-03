---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.2
kernelspec:
  language: julia
  name: julia-1.12
  display_name: Julia 1.12
---

# 第6回: 内心と角の二等分線 — 計量が要る場面（重心のアフィンとの対比）

**主題**: 五心の次の一つ、内心 (Incenter) を **角の二等分線** から組み立てる。前回の重心が **アフィン（斜行で不変）** な点だったのに対し、内心は **計量（角度・距離）に依存** する点である。

**副題**: 同じ「三本が一点で交わる」でも、重心は剪断しても動かず、内心は動く。**何がアフィンで、何が計量か** を区別する回。

**学習目標**

1. 内心を **角の二等分線の特徴づけ（二等分線上の点は両辺から等距離）** だけから、Euclid 流の命題連鎖で導けるようになる。
2. **重心（アフィン）と内心（計量）の違い** を、剪断（せん断変換）で動くか動かないかで説明できるようになる。
3. 前回の Construction Protocol → DataFrame を **再利用** し、三本の角の二等分線が一点で交わることをデータで検証する。
4. **(伏線継続)** アフィンと計量の緊張は、後の円錐曲線（接線・通径）で本格的に現れる。内心の「等距離」は、後で焦点・準線の「距離の比」へ繋がる。

+++

## 0. 前回 (第 5 回) の振り返り — そして今日の対比

前回は **重心** を Archimedes の天秤（細片を中線に沿って釣り合わせる）で導き、それが **アフィンに正当**（斜行でも中線が平行弦を二等分するから、直交は要らない）であることを見た。バリセントリック座標が許容されるのも、それがアフィン不変だからだった。

今日の内心は、そこと **対照的** である。内心は「三辺から等距離の点」「角の二等分線の交点」として定義される。**等距離も角度も、剪断（斜めに引き伸ばす変換）で保たれない** —— つまり内心は **計量的（metric）な点** である。重心（アフィン）と内心（計量）を並べることで、「何が座標選択に依らず、何が依るか」 の感覚を育てる。

```{note} 目標の確認 —— @Codex で達成目標を引く
:class: note
達成目標を **@Codex に lancedb-rag（教材ドメイン RAG）で確認**してもらえる（L5 から運用）。
（プロンプト例）@Codex この回（L6 内心）の達成目標を lancedb-rag で調べて、要点を整理して。
特に「重心はアフィン・内心は計量」 の対比が腑に落ちるか、自分の理解と照らしてから先へ進む。
:::
```

+++

## 1. 内心への道 — 角の二等分線

内心は次の一つの特徴づけから出発する：

> **角の二等分線の特徴づけ**: 角 $\angle BAC$ の二等分線上の点 $X$ は、二辺 $AB$, $AC$ から **等距離** にある（逆も成り立つ）。

外心が「弦の垂直二等分線 = 二点から等距離」だったのと同型の構造だが、**対象が「点からの距離」ではなく「直線（辺）からの距離」** に変わっている。点からの距離は座標を回しても保たれる長さだが、ここで効くのは **辺への垂線の足までの距離** —— 角度に依存する量である。

+++

## 2. ggblab セットアップと三角形の準備

```{code-cell}
using Pkg
Pkg.activate("../..")   # この教材プロジェクトの環境を有効化
Pkg.resolve()
Pkg.instantiate()       # 必要なパッケージを用意（初回は少し時間がかかる）
using GeoGebra
ENV["GGB_DIRECT_TRANSPORT"] = "true"   # ggblab とアプレットの直接通信を有効化
```

```{code-cell}
inject_applet()
```

```{code-cell}
@ggb :const :new
```

```{code-cell}
@ggb A=(0, 0)
@ggb B=(6, 0)
@ggb C=(1, 4)
@ggb Polygon(:A, :B, :C)
```

```{note} 記法メモ —— `@ggb` と `:A` のコロン（一度だけ）
:class: note
`@ggb` は ggblab の **マクロ**で、続く式を GeoGebra アプレットへのコマンドに変換する。`:A` のコロン付きは Julia の **シンボル**（「A という名前そのもの」を指す印）で、**すでに作図したオブジェクトを名前で参照する**ときに使う。読み分けは単純で、`A=(0,0)` のように新しく作るときは左辺コロン無し、`Polygon(:A, :B, :C)` のように既存の点を参照するときはコロン付き（L4/L5 と同じ作法）。
```

## 3. 解くべき命題を発見する

三頂点の内角の二等分線を引いて観察する（規律: 各オブジェクトを一意のシンボルに、入れ子にしない）。

```{code-cell}
@ggb wa = AngleBisector(:B, :A, :C)   # 頂点 A の内角の二等分線
@ggb wb = AngleBisector(:A, :B, :C)   # 頂点 B の内角の二等分線
@ggb wc = AngleBisector(:A, :C, :B)   # 頂点 C の内角の二等分線
```

**観察**: 三本の角の二等分線が一点で交わる。

**問い**: A, B, C の取り方を変えても成り立つか？ 成り立つとしたら、なぜか？ —— L4（外心）・L5（重心）と同じ「観察から命題への移行」 である。

+++

## 4. 証明（Euclid 流の三段）

**Lemma（角の二等分線の特徴づけ）**: 角 $\angle BAC$ の二等分線上の点 $X$ から二辺 $AB$, $AC$ へ下ろした垂線の足までの距離は等しい。

*証明*: $X$ から $AB$, $AC$ への垂線の足を $P$, $Q$ とする。$\triangle AXP$ と $\triangle AXQ$ は、$\angle PAX = \angle QAX$（二等分）、$\angle APX = \angle AQX = 90°$、$AX$ 共通 → 合同（AAS）。ゆえに $XP = XQ$。$\blacksquare$

**主命題**: 三角形 $\triangle ABC$ の三つの内角の二等分線は一点 $I$ で交わり、$I$ は三辺から等距離にある。

*証明*:
1. $A$ と $B$ の二等分線 $w_a$, $w_b$ の交点を $I$ とする。
2. $I \in w_a$ より、$I$ は辺 $AB$ と $AC$ から等距離（Lemma）。
3. $I \in w_b$ より、$I$ は辺 $AB$ と $BC$ から等距離。
4. 2, 3 より $I$ は辺 $AC$ と $BC$ からも等距離 → $I$ は $C$ の二等分線 $w_c$ 上にもある。
5. ゆえに三本の二等分線は一点 $I$ で交わる。
6. $I$ から三辺への等距離を半径とする円は三辺に接する（**内接円**）。$\blacksquare$

```{important} 定義
この交点 $I$ を三角形 $\triangle ABC$ の **内心 (Incenter)**、$I$ を中心とし三辺に接する円を **内接円 (Incircle)** と呼ぶ。
```

+++

```{attention} 重心はアフィン、内心は計量 —— 今日の構造的対比
:class: attention

L5 の重心と L6 の内心は、どちらも「三本が一点で交わる」 が、**本質的に別の種類の点** である:

| | 重心（L5） | 内心（L6） |
|---|---|---|
| 定義 | 中線（頂点と対辺中点）の交点 | 角の二等分線（= 二辺から等距離）の交点 |
| 使う量 | 中点・面積・モーメント（**アフィン量**） | 角度・辺への距離（**計量量**） |
| 剪断（斜め引き伸ばし）で | **動かない**（位置の比が不変） | **動く**（角度・距離が変わる） |
| 座標 | バリセントリックで $(1{:}1{:}1)$、座標選択に依らず | 辺の長さ $a,b,c$ で重みづけ $(a{:}b{:}c)$、計量に依存 |

→ **内心のバリセントリック座標は $(a : b : c)$**（対辺の長さ）であり、重心の $(1:1:1)$ と違って **三角形の計量（辺長）が入る**。だから内心は「斜行で不変」 ではない。L5 で「アフィンの自由」 を使えたのは重心がアフィンだったからで、内心ではその自由は使えない —— **計量が要る**。

この「アフィン量で済むか、計量が要るか」 の区別は、後の円錐曲線で決定的になる。接線（局所構造）や焦点・準線は計量とアフィンの境界に立つ（L11+ で回収）。
```

+++

### 4 補足: 剪断で「内心は動く / 重心は動かない」を作図で確かめる

上の表の「剪断で動くか」の行を、実際に作図で見る。L5 で重心に使った `ApplyMatrix`（せん断行列 $S=\begin{pmatrix}1&k\\0&1\end{pmatrix}$, $\det S = 1$ ゆえ面積保存）を、今日は内心に当てる。**重心は「せん断してから中心を取る」＝「中心を取ってからせん断」が一致した**が、**内心は一致しない** —— これが「内心は計量の点」 の作図による証拠であり、必修課題 2 の足場である。

```{code-cell}
# せん断 S（det = 1, 面積保存）を三角形 ABC に適用（L5 §4.1 と同じ道具）
@ggb k = Slider(-2, 2)
@ggb S = {{1, k}, {0, 1}}
@ggb A_s = ApplyMatrix(:S, :A)
@ggb B_s = ApplyMatrix(:S, :B)
@ggb C_s = ApplyMatrix(:S, :C)
@ggb tri_s = Polygon(:A_s, :B_s, :C_s)
```

```{code-cell}
# 内心を二通りで導いて並記する（規律1: 入れ子にせず中間シンボルを置く）
#   ルート a = TriangleCenter(..., 1) / ルート b = 角の二等分線の交点 Intersect

# 元の三角形 ABC の内心（ルート b は §3 の二等分線 wa, wb を再利用）
@ggb I_tc  = TriangleCenter(:A, :B, :C, 1)   # ルート a
```

```{code-cell}
# せん断後の三角形 A_s B_s C_s の内心（ルート b は二等分線を引き直して交点を取る）
@ggb wa_s    = AngleBisector(:B_s, :A_s, :C_s)
@ggb wb_s    = AngleBisector(:A_s, :B_s, :C_s)
@ggb wc_s    = AngleBisector(:A_s, :C_s, :B_s)
# @ggb I_s_tc  = TriangleCenter(:A_s, :B_s, :C_s, 1)   # ルート a
@ggb I_s_int = Intersect(:wa_s, :wb_s)               # ルート b

# @ggb I_int = Intersect(:wa, :wb)             # ルート b（a と同じ点になるはず）
```

```{code-cell}
# 「元の内心をせん断で移した点」(これとせん断後の内心がずれる = 計量の証拠)
@ggb I_byS = ApplyMatrix(:S, :I_tc)
```

```{code-cell}
# 対比: 重心はアフィンなので「せん断してから取る」=「取ってからせん断」が一致する
@ggb G_orig = TriangleCenter(:A, :B, :C, 2)        # 元の三角形の重心
@ggb G_s    = TriangleCenter(:A_s, :B_s, :C_s, 2)  # せん断後の重心
@ggb G_byS  = ApplyMatrix(:S, :G_orig)             # 元の重心を同じせん断で移した点
```

スライダ $k$ を動かすと、内心はルート a（TriangleCenter）でもルート b（二等分線の交点）でも **同じ点**になる（$I_{tc}=I_{int}$、せん断後も $I_{s\_tc}=I_{s\_int}$）—— 導出経路が違っても内心は一つ。だが、**せん断後の内心 $I_s$ と「元の内心をせん断した点」$I_{byS}$ はずれる**。一方、**重心は $G_s$ と $G_{byS}$ が常に重なる**（アフィン共変 = せん断と中心取りが可換）。せん断は面積（$\det S = 1$）を保つが角度・距離を変えるので、「角の二等分（＝計量）」 で決まる内心は写り先がずれ、中点・モーメント（＝アフィン）で決まる重心はずれない。これが今日の主題 —— **重心はアフィン、内心は計量** —— の、目で見える証拠である。

+++

## 5. ggblab_extra 再利用 — DataFrame で「一点で交わる」を検証する

L5 で導入した Construction Protocol → DataFrame を、今日は **再利用** する。三本の角の二等分線の交点が本当に一点か、データで確かめる。

```{code-cell}
@ggb I = Intersect(:wa_s, :wb_s)          # 内心（規律1: 入れ子にしない）
@ggb I2 = Intersect(:wb_s, :wc_s)         # 別の二本の交点
@ggb I3 = Intersect(:wa_s, :wc_s)         # さらに別の二本
```

```{code-cell}
# 内接円も構成して、三辺に接することを確認する
@ggb c_AB = Line(:A, :B)              # 辺 AB の直線
@ggb r = Distance(:I, :c_AB)          # I から辺への距離 = 内接円半径
@ggb incircle = Circle(:I, :r)
```

```{code-cell}
# Construction Protocol を DataFrame として外部化する準備（L5 §5 と同じ作法）
# Julia の中から Python の道具（ggblab_extra / polars）を借りる ── これが PythonCall
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
# 現在の作図状態を DataFrame に取り出し、三つの交点 I, I2, I3 を並べて比べる
df = @await ConstructionIO.initialize_dataframe(ggb, use_applet=true)
df_I = df.filter(pl.col("Name").is_in(["I", "I2", "I3"]))   # polars の絞り込み構文
df_I
```

```{code-cell}
pl.Config.set_fmt_str_lengths(100)
df_I["Value"]
```

**DataFrame で見ると**: `I`, `I2`, `I3` の `Value` 列が（数値誤差の範囲で）一致 → **三本が同じ点で交わる** ことがデータで確認できる。L5（重心）と **同じ検証の型** を、別の中心で再演している。「内部状態を検査可能なデータに外部化する」 作法が回をまたいで定着する。

```{tip} 進捗の確認 —— セル出力を @Codex に読ませる
:class: tip
@Codex は **jupyter-server-mcp であなたのセル出力を直接読む**（プロンプトだけの対話ではない）。
（プロンプト例）@Codex ここまでのセル出力を読んで、達成目標①（内心を二等分線から導く）③（DataFrame で一点交差を検証）にどこまで到達したか、まだ埋まっていないセルはどこか、具体的に挙げて。
```

+++

## 6. 停滞と画期の照射 — 内心と傍心、そして接線への伏線

内角の二等分線が内心を生むなら、**外角の二等分線** は何を生むか？ —— 一つの内角と他の二つの外角の二等分線は **傍心 (Excenter)** で交わる。日本の幾何教育の「五心」 は、この傍心を主要な中心に数える独自の伝統である（欧米標準では別格扱い）。

内心と傍心は、三角形の辺について **対称的に配置される**。この対称性は教科書では強調されないが、後の回（L9）で **学生自身が発見する第一の山** になる。そしてさらに先（L12–L13）で、この内心・傍心の対称性が **Dandelin 球による楕円二焦点の対称性の二次元版** であることが見えてくる（discussion_memo §10（MCP `lancedb-rag`「内心 傍心 対称 Dandelin 二焦点」project=conversations））。今日の内心は、その長い伏線の起点である。

```{note} 伏線 —— 「等距離」から「距離の比」へ、計量とアフィンの境界
:class: note
内心の核は「**三辺から等距離**」 だった。後の円錐曲線では、焦点・準線の「**距離の比が一定（離心率）**」（Pappus）へと一般化される。等距離（比 = 1）はその特殊な場合と読める。

そして接線（円錐曲線の局所構造）は、L5 で見た **アフィンの自由（Apollonius の斜交直径）** と、本日の **計量** が交わる場所である。Apollonius は接線を斜行座標で静的に書いて封印した（discussion_memo §4（MCP `lancedb-rag`「Apollonius 通径 接線封印 斜交直径」project=conversations））。その封印を解く現代の鍵 —— **接線 = 重解 = 判別式 $D=0$**（SymPy で言語化）—— は、L11+ の円錐切断で回収する。重心（アフィン）→ 内心（計量）→ 接線（両者の緊張）という順序で、二千年の物語に近づいている。

ここに出てくる Dandelin・離心率・通径・準線といった語は、**今は分からなくて正常**。L11+ で必ず戻ってくる。今日は「内心 ＝ 三辺から等距離」 が後で「距離の比」 へ広がる、という**見取り図だけ**持って先へ進めばよい。
```

+++

```{admonition} 今回の課題
:class: tip

**必修**
1. 三角形を一つ作図し、三本の内角の二等分線の交点 $I$（内心）と内接円を構成せよ。Construction Protocol を DataFrame に取り出し、三本の交点が一点であることを `Value` 列で確認せよ。
2. 三角形を **剪断（せん断変換）** して横に倒したとき、重心は中線の交点のまま（アフィン不変）だが、内心は辺への等距離が崩れて **別の点に動く** ことを、作図で確かめよ。

**思考課題**
3. 「等距離（内心）」 と「距離の比が一定（離心率・焦点準線）」 は、どちらも距離で点を定義する。両者の関係を、現時点の自分の言葉で書き留めよ（第 11–15 回での立論の素材）。
```

+++

:::{important} 授業末尾の自己評価 —— `@Codex` に聞いてみる（任意、L4/L5 から継続）

L4/L5 と同じ template で、本回の自己評価を試してください。**任意**です。@Codex は **jupyter-server-mcp で全セルの入出力を読み**、lancedb-rag で lesson 文脈を参照して個別評価を返します。

````text
@Codex 今日のノートブックを全 cell 読んで評価してください。
次の三つを区別して articulate してください:

1. 自分で考えて書いた cell — 思考の痕跡が残っている部分
2. AI 委託で書いたが、理解して受け入れた cell — 動いて、なぜ動くか説明できる部分
3. AI 委託で書いたが、なぜ動くか説明できない cell — 動いているが、理解で未到達の部分

加えて: 今日の主題（内心 / 角の二等分線 / 重心アフィン vs 内心計量 / DataFrame 再利用）
に対する到達度、完成しないまま残った問い、次回（L7 垂心・Euler 線）への接続点。

特に §4 の「重心はアフィン、内心は計量」 対比について、剪断で内心が動くことを
自分が腑に落とせたか、誤魔化さず正直に articulate してください。
````

JupyterAI のやり取り log は LMS 経由で先生に届きます —— 学期末立論（第 15 回）の materials として毎週蓄積。「**先週は気付かなかったことに今週は気付けた**」 が回を重ねるごとに起こるか、皆さん自身でも観察してください。
:::

+++

## 7. 次回への接続

次回（第 7 回）は **垂心 (Orthocenter) と Euler 線** に進む。外心・重心・垂心の三つが一直線上に並ぶ（Euler 線）という事実は、座標計算では「偶然」 に見える。そこで Construction Protocol → DataFrame を一歩進めて **有向グラフ (networkx)** に変換し、三中心が **共通の前提から派生する依存構造** を持つことを見る（ggblab_extra 機能の第二段、ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook） L7）。

このとき、本日 §5 で「各オブジェクトを一意のシンボルに束縛する」 規律が効いてくる —— 名前のないオブジェクトはグラフのノードにできないからである（教材オーサリング規約（MCP `lancedb-rag`「教材オーサリング規約 ggblab セル シンボル」project=textbook） 規律 1）。

+++

## 参考文献

- [Incenter - Wikipedia](https://en.wikipedia.org/wiki/Incenter)
- [Angle bisector - Wikipedia](https://en.wikipedia.org/wiki/Bisection#Angle_bisector)
- [Excenter / Excircle - Wikipedia](https://en.wikipedia.org/wiki/Incircle_and_excircles)
- [Barycentric coordinate system - Wikipedia](https://en.wikipedia.org/wiki/Barycentric_coordinate_system)（内心 = $(a:b:c)$、重心 = $(1:1:1)$）
- 前回 前回 第5回(重心)（MCP `lancedb-rag`「第5回 重心 DataFrame アフィン せん断」project=textbook） — 重心（アフィン）、Construction Protocol → DataFrame
- 接線封印の系譜 discussion_memo §4/§10（MCP `lancedb-rag`「Apollonius 通径 接線封印 / 内心傍心 Dandelin」project=conversations）
- 道具のロードマップ ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook） / 著者規約 教材オーサリング規約（MCP `lancedb-rag`「教材オーサリング規約 ggblab セル シンボル」project=textbook）
