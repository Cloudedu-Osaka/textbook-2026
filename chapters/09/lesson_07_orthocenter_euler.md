---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.2
kernelspec:
  display_name: Julia 1.12
  name: julia-1.12
  language: julia
---

# 第7回: 垂心と Euler 線 — 三中心が一直線に並ぶ「隠れた構造」を有向グラフで見る

**主題**: 五心の四つ目、垂心 (Orthocenter) を **頂点から対辺への垂線（高さ）** から組み立てる。そして外心 $O$（L4）・重心 $G$（L5）・垂心 $H$（本日）の **三つが一直線上に並ぶ**（**Euler 線**）という事実に出会う。

**副題**: 座標で計算すると、三中心の共線は **偶然** に見える。だが「どのオブジェクトがどのオブジェクトから作られたか」 の **依存構造（有向グラフ）** で見ると、共線は偶然ではなく **共通の前提から派生する必然** だと分かる。Construction Protocol を **DataFrame の一歩先＝グラフ** へ進める回。

**学習目標**

1. 垂心を **「各頂点から対辺へ下ろした垂線は一点で交わる」** という命題から、L4（外心）の結果に帰着させる形で Euclid 流に導けるようになる。
2. **Euler 線**（$O$, $G$, $H$ が一直線、$OG : GH = 1 : 2$）を、外心を原点に取ったベクトルで確かめられるようになる。
3. Construction Protocol → DataFrame を一歩進め、**有向グラフ (networkx)** へ変換して、三中心が **共通の頂点 $A,B,C$ から派生する依存構造** を持つことを「目で見える形」 にする（ggblab_extra 機能の第二段、DAG 初出）。
4. **(伏線継続・接線方向への一歩)** Euler 線は三角形に隠れていた **一本の直線** だった。後の円錐曲線では、図形に隠れた **軸・準線** という直線が主役になる。「隠れた直線を構造から見つける」 という今日の経験は、その予行演習である。

+++

## 0. 前回 (第 6 回) の振り返り — そして今日の「隠れた直線」

前回は **内心** を角の二等分線（＝二辺から等距離）で導き、それが **計量（角度・距離）に依存** する点であることを、剪断で内心が動くことから確かめた。L5 の重心（アフィン）と L6 の内心（計量）—— 「何が座標選択に依らず、何が依るか」 の対比だった。

ここまでで三つの中心を **別々に** 作ってきた：外心 $O$（弦の垂直二等分線、L4）、重心 $G$（中線、L5）、内心 $I$（角の二等分線、L6）。今日は四つ目の **垂心 $H$**（高さ）を作る。そして —— ここからが今日の主題 —— **外心 $O$・重心 $G$・垂心 $H$ の三つは、どんな三角形でも一直線上に並ぶ**。この直線を **Euler 線** と呼ぶ。

三角形を眺めても、この直線は見えない。座標を入れて計算すると三点が確かに一直線に乗るが、それは「**たまたまそうなった**」 ように見える。今日はこの「たまたま」 を、**依存構造（どの点がどの点から作られたか）の有向グラフ** に外部化して、「偶然ではなく、共通の前提から来る必然」 として読み解く。

```{note} 目標の確認 —— @Codex で達成目標を引く
:class: note
達成目標を **@Codex に lancedb-rag（教材ドメイン RAG）で確認**してもらえる（L5 から運用）。
（プロンプト例）@Codex この回（L7 垂心・Euler 線）の達成目標を lancedb-rag で調べて、要点を整理して。
特に「三中心の共線は座標では偶然に見えるが、依存グラフでは必然」 という主張が腑に落ちるか、自分の理解と照らしてから先へ進む。
:::
```

+++

## 1. 垂心への道 — 頂点から対辺への垂線（高さ）

垂心は次の一つの命題から出発する：

> **垂線（高さ）の共点性**: 三角形 $\triangle ABC$ の各頂点から対辺へ下ろした垂線（**高さ**）は、一点で交わる。

L4（外心 = 弦の垂直二等分線の交点）、L6（内心 = 角の二等分線の交点）と同じ「三本が一点で交わる」 型だが、今日の三本は **垂線（高さ）** である。注目したいのは、この共点性が **新しい証明を要しない** こと —— うまく補助線を引くと、**L4（外心）の結果にそのまま帰着する**。「既に証明したことに帰着させる」 のは、Euclid 流の命題連鎖の核である。

+++

## 2. ggblab セットアップと三角形の準備

```{code-cell}
using Pkg
Pkg.activate("../..")   # この教材プロジェクトの環境を有効化
Pkg.resolve()
Pkg.instantiate()       # 必要なパッケージを用意（初回は少し時間がかかる）
```

```{code-cell}
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
```

```{code-cell}
@ggb B=(6, 0)
@ggb C=(1, 4)
@ggb Polygon(:A, :B, :C)
```

:::{note} 記法メモ —— `@ggb` と `:A` のコロン（L4–L6 と同じ作法）
:class: note
新しく作るときは左辺コロン無し（`A=(0,0)`）、既存の点を **名前で参照** するときはコロン付きシンボル（`Line(:B, :C)`）。そして本日の鍵となる **規律1**: 各オブジェクトを **一意のシンボル** に束縛し、**入れ子にしない**（`Intersect(Line(...), Line(...))` のように中間を名無しにしない）。理由は §5 で効いてくる —— **名前のないオブジェクトは、依存グラフのノードにできない** からである。
:::

+++

## 3. 解くべき命題を発見する

各頂点から対辺へ垂線を下ろし、さらに外心・重心も置いて観察する（規律1: 各オブジェクトを一意のシンボルに、入れ子にしない）。

```{code-cell}
# 対辺の直線（高さを下ろす相手）
@ggb a_BC = Line(:B, :C)   # 頂点 A の対辺
@ggb b_CA = Line(:C, :A)   # 頂点 B の対辺
@ggb c_AB = Line(:A, :B)   # 頂点 C の対辺
```

```{code-cell}
# 各頂点から対辺への垂線（高さ）
@ggb ha = PerpendicularLine(:A, :a_BC)
@ggb hb = PerpendicularLine(:B, :b_CA)
@ggb hc = PerpendicularLine(:C, :c_AB)
```

```{code-cell}
# 三つの中心を、それぞれ別経路で置く（規律1: 入れ子にしない）
@ggb O = TriangleCenter(:A, :B, :C, 3)   # 外心（L4）
@ggb G = TriangleCenter(:A, :B, :C, 2)   # 重心（L5）
@ggb H = Intersect(:ha, :hb)             # 垂心 = 高さ二本の交点（本日）
```

**観察1**: 三本の高さ $h_a, h_b, h_c$ が一点 $H$ で交わる（`Intersect(:ha, :hc)` も同じ点になる）。

**観察2**: 外心 $O$・重心 $G$・垂心 $H$ の三点が **一直線** 上に並ぶ。

**問い**: $A, B, C$ の取り方を変えても、この二つは成り立つか？ 成り立つなら、なぜか？ —— L4–L6 と同じ「観察から命題への移行」 である。

+++

## 4. 証明（既知への帰着 + ベクトル）

### 4.1 垂線の共点性 —— 外心 (L4) へ帰着する

**主命題1**: 三角形 $\triangle ABC$ の三つの高さは一点 $H$ で交わる。

*証明*: 各頂点を通り、対辺に **平行な** 三直線を引く。それらで囲まれる大きな三角形を $\triangle A'B'C'$ とする（$A$ は辺 $B'C'$ の中点、$B$ は $C'A'$ の中点、$C$ は $A'B'$ の中点になる）。

- $BC \parallel B'C'$ であり、$A$ から $BC$ への高さ $h_a$ は $B'C'$ にも垂直。さらに $A$ は $B'C'$ の中点。ゆえに $h_a$ は **辺 $B'C'$ の垂直二等分線** である。
- 同様に $h_b$, $h_c$ は $\triangle A'B'C'$ の辺 $C'A'$, $A'B'$ の垂直二等分線になる。
- 三辺の垂直二等分線は一点で交わる —— これは **L4 で証明した外心の共点性** そのもの。

ゆえに $\triangle ABC$ の三つの高さは、$\triangle A'B'C'$ の外心という一点で交わる。$\blacksquare$

```{important} 定義
この交点 $H$ を三角形 $\triangle ABC$ の **垂心 (Orthocenter)** と呼ぶ。$\triangle ABC$ の垂心は、頂点を通り対辺に平行な線で囲んだ大三角形 $\triangle A'B'C'$ の **外心** に等しい。
```

### 4.2 Euler 線 —— 外心を原点に取る

**主命題2（Euler, 1765）**: 外心 $O$・重心 $G$・垂心 $H$ は一直線上にあり、$OG : GH = 1 : 2$（すなわち $\overrightarrow{OH} = 3\,\overrightarrow{OG}$）。

*証明*: 外心 $O$ を原点に取る。$\mathbf a = \overrightarrow{OA}$, $\mathbf b = \overrightarrow{OB}$, $\mathbf c = \overrightarrow{OC}$ とすると、外心の定義より $|\mathbf a| = |\mathbf b| = |\mathbf c| = R$（外接円半径）。

ここで点 $H$ を $\overrightarrow{OH} = \mathbf a + \mathbf b + \mathbf c$ で定める。この $H$ が垂心であることを示す：
$$\overrightarrow{AH}\cdot\overrightarrow{BC} = (\mathbf b + \mathbf c)\cdot(\mathbf c - \mathbf b) = |\mathbf c|^2 - |\mathbf b|^2 = R^2 - R^2 = 0.$$
ゆえに $AH \perp BC$、つまり $H$ は $A$ からの高さ上にある。三頂点で同様だから、$H = \mathbf a + \mathbf b + \mathbf c$ は三つの高さの交点 = 垂心。

一方、重心は $\overrightarrow{OG} = \dfrac{\mathbf a + \mathbf b + \mathbf c}{3} = \dfrac{1}{3}\overrightarrow{OH}$。

よって $O$（原点）, $G$, $H$ は同一直線上にあり、$OG : GH = 1 : 2$。$\blacksquare$

```{attention} 三つの中心を一枚に並べる —— Euler 線という「隠れた直線」
:class: attention

L4–L7 で作った四つの中心を、性質で並べる：

| 中心 | 定義 | L | Euler 線上か |
|---|---|---|---|
| 外心 $O$ | 弦の垂直二等分線（三点から等距離） | L4 | ○ |
| 重心 $G$ | 中線（アフィン量） | L5 | ○（$OH$ を $1:2$ に内分） |
| 垂心 $H$ | 高さ（外心へ帰着） | L7 | ○ |
| 内心 $I$ | 角の二等分線（計量量） | L6 | **×**（一般には乗らない） |

→ 三つ（$O, G, H$）は一直線（Euler 線）に乗るが、**内心 $I$ は乗らない**。なぜ内心だけ外れるのか —— $O, G, H$ はいずれも頂点 $\mathbf a, \mathbf b, \mathbf c$ の **一次結合（重み和）** で書けるが、内心は $(\,a\mathbf a + b\mathbf b + c\mathbf c\,)/(a{+}b{+}c)$ と **辺長 $a,b,c$（計量）が重みに入る** ため、線形の物語から外れる。L5–L6 で見た「アフィン量で済むか、計量が要るか」 の区別が、ここでも効いている。

そして Euler 線は、**三角形を眺めても見えない直線** だった。それを「中心たちの一次結合」 という構造から取り出した。図形に隠れた直線を構造から見つける —— この経験は、後の円錐曲線で **軸・準線** という隠れた直線を扱うときの足場になる（L10+ で回収）。
```

+++

## 5. ggblab_extra 第二段 —— DataFrame を「有向グラフ」へ進める

L5–L6 では Construction Protocol を **DataFrame**（表）に外部化した。今日はもう一歩進めて、それを **有向グラフ (directed graph)** に変換する。各オブジェクトを **ノード**、「Y は X から作られた」 という依存関係を **矢印（X → Y）** にする。狙いは、$O, G, H$ が **共通の頂点 $A, B, C$ から派生している** ことを「目で見える形」 にして、Euler 共線が偶然でないことを構造として示すことである。

```{code-cell}
# L5–L6 と同じ作法で、現在の作図状態を DataFrame に取り出す（Julia から Python の道具を借りる）
using PythonCall
pl = pyimport("polars")
nx = pyimport("networkx")            # 本日の新顔: 有向グラフの道具
ggb = pyimport("ggblab")
ggblab_extra = pyimport("ggblab_extra")
ggb.file = pyimport("ggblab.file").ggb_file()
ggb.parser = pyimport("ggblab.parser").ggb_parser()
ggb.schema = pyimport("ggblab.schema").ggb_schema()
ConstructionIO = ggblab_extra.ConstructionIO
```

```{code-cell}
# 作図全体を DataFrame に。各行は (Name, Command, Value) を持つ。
df = @await ConstructionIO.initialize_dataframe(ggb, use_applet=true)
df.select(["Name", "Command"])   # 各オブジェクトが「何から作られたか」の Command 列（定義）を見る
```

```{code-cell}
# Command 列に現れる「既存オブジェクト名」を依存関係 (X → Y) として有向グラフに積む。
# ここで規律1（各オブジェクトに一意の名前・入れ子にしない）が効く: 名前があるから矢印を張れる。
using PythonCall
names = [pyconvert(String, n) for n in df["Name"].to_list()]
G_dag = nx.DiGraph()
for row in df.iter_rows(named=true)
    name = pyconvert(String, row["Name"])
    defn = pyconvert(String, row["Command"])
    G_dag.add_node(name)
    for src in names
        # 自分以外の既存名が定義式に現れたら「src → name」の依存辺を張る
        if src != name && occursin(Regex("\\b" * src * "\\b"), defn)
            G_dag.add_edge(src, name)
        end
    end
end
println("ノード数 = ", pyconvert(Int, G_dag.number_of_nodes()),
        " / 辺数 = ", pyconvert(Int, G_dag.number_of_edges()))
```

```{code-cell}
# 三中心 O, G, H それぞれの「祖先」（それを作るのに使われた全オブジェクト）を取り出す。
anc_O = Set(pyconvert(String, x) for x in nx.ancestors(G_dag, "O"))
anc_G = Set(pyconvert(String, x) for x in nx.ancestors(G_dag, "G"))
anc_H = Set(pyconvert(String, x) for x in nx.ancestors(G_dag, "H"))
println("O の祖先: ", sort(collect(anc_O)))
println("G の祖先: ", sort(collect(anc_G)))
println("H の祖先: ", sort(collect(anc_H)))
println("三中心が共有する祖先: ", sort(collect(intersect(anc_O, anc_G, anc_H))))
```

**グラフで見ると**: $O, G, H$ の祖先をたどると、**いずれも頂点 $A, B, C$（と、それらから作った辺・高さ）に収束する**。三中心は別々の経路（垂直二等分線 / 中線 / 高さ）で作られたが、**根は同じ三頂点** —— Euler 共線は「無関係な三点がたまたま並んだ」 のではなく、**同じ前提から派生した三点だから並ぶ**。座標値の一致（DataFrame の `Value` 列）が「結果」 なら、依存グラフ（祖先の共有）は「理由」 を示している。

```{tip} 進捗の確認 —— セル出力を @Codex に読ませる
:class: tip
@Codex は **jupyter-server-mcp であなたのセル出力を直接読む**（プロンプトだけの対話ではない）。
（プロンプト例）@Codex ここまでのセル出力を読んで、達成目標①（垂心を外心へ帰着）②（Euler 線をベクトルで確認）③（依存グラフで三中心の共有祖先を可視化）にどこまで到達したか、まだ埋まっていないセルはどこか、具体的に挙げて。
```

+++

## 6. 停滞と画期の照射 —— 「隠れた直線」と、次の傍心

Euler が 1765 年にこの共線を見つけるまで、外心・重心・垂心は二千年「別々の中心」 だった。三つを一本の直線が貫いていることは、座標幾何（Descartes 以降）で計算すれば確かめられるが、**計算は「並ぶ」 ことを教えても「なぜ並ぶか」 を教えない**。今日 §5 で依存グラフに外部化したのは、その「なぜ」 —— 共通の前提から派生する構造 —— を可視化するためだった。

次回（第 8 回）は **傍心 (Excenter)** に進む。内角の二等分線が内心を生んだなら、**外角の二等分線** は何を生むか。一つの内角と二つの外角の二等分線は傍心で交わり、内心と合わせて **四つの中心** が三角形の辺について対称に配置される。この対称性は、さらに先（L9）で学生自身が発見する山場になり、L12–L13 で **Dandelin 球による楕円二焦点の対称性** の二次元版だと分かる。

```{note} 伏線 —— Euler 線（隠れた直線）から、円錐曲線の「軸・準線」へ
:class: note
今日の Euler 線は、三角形に隠れていた **一本の直線** を、中心たちの一次結合という構造から取り出したものだった。後の円錐曲線では、図形に隠れた直線が再び主役になる：楕円・放物線・双曲線の **対称軸**、そして焦点と対をなす **準線** である。

しかも —— L6 の伏線（「等距離」 → 「距離の比」）とここでつながる —— 準線は「焦点からの距離と準線からの距離の **比** が一定（離心率）」 という形で図形を定義する隠れた直線である。Euler 線を「中心たちの構造から見つけた」 経験は、準線を「円錐曲線の構造から見つける」 ための予行演習になる。

ここで出てくる軸・準線・離心率・Dandelin といった語は、**今は分からなくて正常**。L10+ で必ず戻ってくる。今日は「図形に隠れた直線を、構造から取り出せる」 という**手応えだけ**持って先へ進めばよい。
```

+++

## 発展 —— 直角三角形への退化: 高さから Pythagoras が立ち上がる

§4 で垂心を「高さ三本の交点」として定義した。この垂心を **直角三角形** へ退化させると、二千年前の Pythagoras の定理が、今日の「高さ」の一番素直な特殊化として立ち上がる。

```{attention} 高さ → 等積変形 → Pythagoras —— 受験数学の到達点
:class: attention

**① 垂心が直角の頂点に潰れる。** 直角をはさむ二辺は、それぞれ互いの「高さ」でもある（脚 $\perp$ 脚）。だから二本の高さは直角の頂点で交わり、第三の高さ（直角の頂点から斜辺へ下ろした垂線）もそこを通る。→ **直角三角形の垂心は、直角の頂点に一致する**。一般には三角形の内外を動き回る $H$ が、ここでは一点へ縮む。

**② その「高さ」は、Euclid が Pythagoras を証明した補助線そのもの。** 原論 I.47（風車の図）は、直角の頂点から斜辺へ垂線を下ろし、それを斜辺の正方形まで延長して、正方形を二つの長方形に分ける。今日 §4 で引いた **高さ** が、そのまま二千年前の証明の主役だった。

**③ 証明の心臓は「等積変形」。** 「同じ底辺・平行線の間にある三角形は等積」（I.35–41）を使い、脚の正方形 $\to$ 三角形 $\to$ 平行移動した三角形 $\to$ 斜辺側の長方形、と **面積を保ったまま形を滑らせて** 移す。脚二つの正方形が、斜辺の正方形の二つの長方形にぴたりと収まる —— これが $a^2 + b^2 = c^2$。

つまり **垂心（計量概念）の最も退化した姿が、等積変形（アフィン操作）で Pythagoras を成立させる**。§4 末尾の囲み（Euler 線の表）で立てた「アフィン量で済むか、計量が要るか」の対比が、ここでも顔を出す —— 直角という **計量** から入り、保存則は **アフィン** で回す。L5 の天秤（斜行細片の積分でアフィンに正当）と同じ呼吸である。

そして等積変形は、**受験数学の到達点**。その到達点が、今日の「高さ」の最も素直な特殊化として目の前に立ち上がる。
```

```{code-cell}
# 直角三角形を一つ作る（直角は P）。§2–§3 の一般の三角形とは別物なので作図をまっさらにする。
# （§5 の Euler-DAG はこの上のセルで済んでいるので、ここで :new してよい）
@ggb :const :new
@ggb P=(0, 0)
@ggb Q=(4, 0)
@ggb R=(0, 3)
@ggb Polygon(:P, :Q, :R)
```

```{code-cell}
# 垂心を直接置く（TriangleCenter ..., 4 = 垂心）。直角の頂点 P に重なるはず。
@ggb Hr = TriangleCenter(:P, :Q, :R, 4)
@ggb d_HrP = Distance(:Hr, :P)   # 0 になる ← 直角三角形の垂心は直角の頂点
```

```{code-cell}
# 直角の頂点 P から斜辺 QR へ下ろした垂線 = Euclid I.47 の補助線そのもの。
@ggb hyp = Line(:Q, :R)                 # 斜辺
@ggb alt = PerpendicularLine(:P, :hyp)  # 高さ（= 補助線）。延ばすと斜辺の正方形を二分する
@ggb Foot = Intersect(:alt, :hyp)       # 垂線の足（斜辺の正方形を二つの長方形に分ける起点）
```

```{code-cell}
# 三辺の上に外向きの正方形を立てる（Pythagoras の図）。
# 規約: Polygon(X, Y, 4) は有向辺 X→Y の「左側」に正方形を作る。向きを選んで外側へ出す。
@ggb sqPQ = Polygon(:Q, :P, 4)   # 脚 PQ の上（下向き＝外）
@ggb sqPR = Polygon(:P, :R, 4)   # 脚 PR の上（左向き＝外）
@ggb sqQR = Polygon(:R, :Q, 4)   # 斜辺 QR の上（原点と反対側＝外）
```

**読み方**: 高さ `alt` を斜辺の正方形 `sqQR` まで延ばすと、正方形が `Foot` を境に **二つの長方形** に分かれる。等積変形（同じ底辺・平行線間の三角形は等積）で滑らせると、**左の長方形 = 脚 `sqPR` の正方形**、**右の長方形 = 脚 `sqPQ` の正方形** とそれぞれ等積になる。脚二つの正方形が斜辺の正方形に過不足なく収まる —— これが $|PQ|^2 + |PR|^2 = |QR|^2$。今日の「高さ」が、受験幾何の華である等積変形を経由して、Pythagoras に直結している。

+++

```{admonition} 今回の課題
:class: tip

**必修**
1. 三角形を一つ作図し、三本の高さの交点 $H$（垂心）を構成せよ。さらに外心 $O$・重心 $G$ を置き、三点を通る直線（Euler 線）を引いて、$G$ が線分 $OH$ を $1:2$ に内分すること（`Distance` で $OG$ と $GH$ を測る）を確かめよ。
2. Construction Protocol を DataFrame に取り出し、**依存関係を有向グラフ (networkx)** に変換せよ。$O, G, H$ の **共有する祖先** に頂点 $A, B, C$ が含まれることを `nx.ancestors` で示せ。

**思考課題**
3. 内心 $I$ も置いて、$I$ が Euler 線に **乗らない** ことを作図で確かめよ。$O, G, H$ は乗るのに内心だけ外れる理由を、§4 の「一次結合 vs 辺長で重みづけ（計量）」 を手がかりに、自分の言葉で書き留めよ（L5–L6 の「アフィン vs 計量」 の延長線、第 10–15 回の立論の素材）。
```

+++

:::{important} 授業末尾の自己評価 —— `@Codex` に聞いてみる（任意、L4–L6 から継続）

L4–L6 と同じ template で、本回の自己評価を試してください。**任意**です。@Codex は **jupyter-server-mcp で全セルの入出力を読み**、lancedb-rag で lesson 文脈を参照して個別評価を返します。

````text
@Codex 今日のノートブックを全 cell 読んで評価してください。
次の三つを区別して articulate してください:

1. 自分で考えて書いた cell — 思考の痕跡が残っている部分
2. AI 委託で書いたが、理解して受け入れた cell — 動いて、なぜ動くか説明できる部分
3. AI 委託で書いたが、なぜ動くか説明できない cell — 動いているが、理解で未到達の部分

加えて: 今日の主題（垂心 / 外心へ帰着する証明 / Euler 線 / DataFrame → 有向グラフ）
に対する到達度、完成しないまま残った問い、次回（L8 傍心）への接続点。

特に §5 の「依存グラフで三中心の共有祖先が見える」 が、Euler 共線を
「偶然」 から「構造的必然」 へ読み替える助けになったか、誤魔化さず正直に articulate してください。
````

JupyterAI のやり取り log は LMS 経由で先生に届きます —— 学期末立論（第 15 回）の materials として毎週蓄積。「**先週は気付かなかったことに今週は気付けた**」 が回を重ねるごとに起こるか、皆さん自身でも観察してください。
:::

+++

## 7. 次回への接続

次回（第 8 回）は **傍心 (Excenter)** に進む。**外角の二等分線** が作る三つの傍心と、L6 の内心とを合わせた **四つの中心** の、辺に関する **対称的な配置** を見る。道具としては、本日導入した **DataFrame → 有向グラフ** を **復習** し（ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook） L8）、内心と傍心が依存構造の上で「鏡像」 の関係にあることを確かめる。

そして本日の規律1（各オブジェクトを一意のシンボルに束縛し、入れ子にしない）が、引き続き効く —— 名前のないオブジェクトは、グラフのノードにできないからである（教材オーサリング規約（MCP `lancedb-rag`「教材オーサリング規約 ggblab セル シンボル」project=textbook） 規律1）。

+++

## 参考文献

- [Orthocenter / Altitude - Wikipedia](https://en.wikipedia.org/wiki/Altitude_(triangle))
- [Euler line - Wikipedia](https://en.wikipedia.org/wiki/Euler_line)（$O, G, H$ 共線・$OG:GH=1:2$）
- [Triangle center - Wikipedia](https://en.wikipedia.org/wiki/Triangle_center)（Kimberling 番号: $X_2$ 重心, $X_3$ 外心, $X_4$ 垂心）
- [NetworkX documentation](https://networkx.org/)（`DiGraph`, `ancestors`）
- 前回 第6回(内心)（MCP `lancedb-rag`「第6回 内心 角の二等分線 計量 アフィン」project=textbook） — 内心（計量）、Construction Protocol → DataFrame
- 隠れた直線と対称の系譜 discussion_memo（MCP `lancedb-rag`「内心傍心 Dandelin 二焦点 対称 / 軸 準線 離心率」project=conversations）
- 道具のロードマップ ggblab_extra ロードマップ（MCP `lancedb-rag`「ggblab_extra 機能配置 ロードマップ」project=textbook） / 著者規約 教材オーサリング規約（MCP `lancedb-rag`「教材オーサリング規約 ggblab セル シンボル」project=textbook）
