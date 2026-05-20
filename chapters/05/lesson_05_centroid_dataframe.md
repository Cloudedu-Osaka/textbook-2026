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
  language: julia
  name: julia-1.12
---

# 第5回: 同じ点を三つの前提から導く — 重心と DataFrame 外部化

**主題**: 重心 (Centroid) は外心とは違い、円の定義からは出てこない。代わりに **三つの異なる前提** から到達される。

**副題**: 三つの前提が「同じ点」に至ることを保証するには何が必要か。

**学習目標**

1. 重心を三つの異なる前提（Archimedes の天秤・面積分割・バリセントリック座標）から導けるようになる。
2. 作図履歴を Construction Protocol → DataFrame として **外部化** することの意味を理解する。
3. **同じ点に到達した** ことを、座標値ではなくデータ構造で検査できるようになる。

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
```

```{code-cell}
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
```

```{code-cell}
@ggb Segment(:A, :M_a)
@ggb Segment(:B, :M_b)
@ggb Segment(:C, :M_c)
```

**観察**: 三本の中線が一点で交わる。さらに、その点は中線を **2:1 に内分** しているように見える。

**問い**: B, C の取り方を変えても成り立つか？ 成り立つとしたら、なぜか？

前回と同じ構造の問い — 観察から命題への移行です。

+++

## 4. 三つの前提からの証明

三つの道で同じ点に到達できることを順に確認します。

+++

### 4.1 面積分割による証明（Euclid 流）

1. 中線 $AM_a$ は、$\triangle ABM_a$ と $\triangle ACM_a$ に三角形を分ける。
2. 両者は底辺 $BM_a$, $CM_a$ が等しく（中点の定義）、高さが共通（A から底辺への垂線）。  
   → **二つの三角形は等面積**。
3. 同様に、中線 $BM_b$, $CM_c$ もそれぞれ三角形を等面積に分ける。
4. 三本の中線が同じ点 G で交わると仮定して、面積比から AG:GM_a を計算すると **2:1** になる。
5. （詳細な作図は付録参照）

参考: [Centroid - Wikipedia](https://en.wikipedia.org/wiki/Centroid#Properties)

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

**ポイント**: この証明は座標系の取り方に依存しない。バリセントリック座標は、点の表現を「アフィン不変」な形にしている。  
→ Möbius (1827) の発想は、座標選択の自由を構造的に吸収する画期。

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

### 4.3 Archimedes の天秤による証明

1. A, B, C それぞれに同じ質量 $m$ を置く。
2. B と C に置いた質量 $m, m$ は、その中点 $M_a$ に集中された質量 $2m$ と平衡する（天秤の原理）。
3. A の質量 $m$ と $M_a$ の質量 $2m$ は、線分 $AM_a$ 上の点 G で平衡する。
4. 平衡条件 $m \cdot AG = 2m \cdot GM_a$ より、**AG : GM_a = 2 : 1**。
5. 同じ議論を B, C を起点として繰り返すと、三本の中線がすべて G で交わることが分かる。

**ポイント**: 物理直観 (天秤) だけで結論に到達する。座標も等式も不要。  
→ Archimedes は紀元前 3 世紀にこの議論を行った。

+++

```{important} 発見
三つの異なる前提（面積分割・バリセントリック・天秤）が、**同じ点** $G$ に到達した。AG:GM_a = 2:1 も三つすべてで一致。

これは偶然ではなく、三つが **同じ構造を別の言語で記述している** ことの帰結である。

では、三つが「同じ点を指している」ことを、どう **検査** すればよいか？
```

+++

```{attention} 三つの証明、それぞれの「納得できなさ」
:class: attention

三つが同じ点に到達した、と書いた。しかし正直に言うと、三つともそれぞれ「完全には納得できない」 ところがある:

- **面積分割** (§4.1) は手順 4 で「三本の中線が同じ点 G で交わると **仮定して**」 から始まる。**一点で交わること自体は別に証明する必要** がある。手順 1-3 だけでは閉じない。
- **天秤** (§4.3) は物理的直観 (連続体の重心と質点の重心が同じ振る舞いをする) を hypothesis として使っている。Archimedes 自身は独立に正当化したが、現代的厳密さからは公理ではなく定理である。
- **バリセントリック** (§4.2) は座標選択を消すが、「**なぜ** 三本の中線が交わるか」 自体は説明していない。手順 3 の「対称性より $α = β = γ$」 は、交わることを暗黙に使っている。

つまり、三つの「証明」 は **異なる暗黙の前提** に依存しており、独立に完結した証明ではない。それでも教育上有益なのは、「同じ結論に至る三つの道筋」 を観察すること自体に意味があるからである。**完全に納得できる証明はおそらく、これら三つを統合した先にある** — そしてそれは線形代数や射影幾何の本格的な道具立てを必要とする (Cauchy/Weierstrass の二千年後の厳密化 §6 を参照)。

これは「証明の隙」 を学生が見つけるための callout でもある。気持ち悪いと思ったら、それは正しい感覚である。L4 で扱った Thales (外心) 証明とは違って、L5 (重心) の証明は **教育の伝統が隙を整理しないまま伝えてきた** instance であり、ここを律儀に articulate することそのものが今日の練習の一つになる。
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
@ggb G_balance = TriangleCenter(:A, :B, :C, 2)  # 2 = Centroid
```

```{code-cell}
# バリセントリック座標による重心
@ggb "G_{bary} = (A + B + C) / 3"
```

```{code-cell}
# 面積分割（中線の交点）による重心
@ggb sa = Segment(:A, :M_a)
@ggb sb = Segment(:B, :M_b)
@ggb G_{medians} = Intersect(:sa, :sb)
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
ggb.file = pyimport("ggblab.file").ggb_file()
ggb.parser = pyimport("ggblab.parser").ggb_parser()
ggb.schema = pyimport("ggblab.schema").ggb_schema()
ConstructionIO = ggblab_extra.ConstructionIO
```

```{code-cell}
df = @await ConstructionIO.initialize_dataframe(ggb, use_applet=true)
```

```{code-cell}
df_centroids = df.filter(pl.col("Name").is_in(["G_balance", "G_bary", "G_medians"]))
df_centroids
```

**DataFrame で見ると、何が起きるか**：

- 三行のうち、`Value` 列の座標が（数値誤差の範囲で）一致している → 三つの起点が同じ点を指していることが **データレベルで確認** できる
- `Command` 列を見ると、三つが **異なる手続き** で構成されていることが分かる → 異なる前提から同じ結論に至った経路が記録されている
- `Type` 列は全て `point` → 三つとも点として扱われている

観察を **目視ではなくデータで検証** できる。これは Apollonius 期の幾何学が手作業で行っていた作業を、現代のツールで一段階上にした行為です。

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

しかし、これが **解析的・座標非依存に厳密化** されたのは、Möbius のバリセントリック座標 (1827) を待たねばなりませんでした。

| 段階 | 何が画期だったか |
|---|---|
| 紀元前 3 世紀 (Archimedes) | 物理直観で重心を **正しく** 計算できた |
| 1637 (Descartes) | 座標を導入したが、重心の本質は座標の選び方に依存しないはず |
| 1827 (Möbius) | バリセントリック座標で **座標選択の自由を構造的に吸収** |
| 1844 (Grassmann) | 線形結合の代数を抽象化（ベクトル空間の先駆） |
| 19 世紀後半 (Gibbs/Heaviside) | ベクトル代数として物理に整備 |

→ **「正しい答え」と「なぜ正しいかが明確な答え」の間に二千年の遅延** があった。

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

道具のロードマップは [`ggblab_extra_roadmap.md`](ggblab_extra_roadmap.md) を参照。

+++

## 参考文献

- [Centroid - Wikipedia](https://en.wikipedia.org/wiki/Centroid)
- [Barycentric coordinate system - Wikipedia](https://en.wikipedia.org/wiki/Barycentric_coordinate_system)
- [August Ferdinand Möbius - Wikipedia](https://en.wikipedia.org/wiki/August_Ferdinand_M%C3%B6bius)
- [Archimedes' On the Equilibrium of Planes (Heath 訳)](https://archive.org/details/worksofarchimede00archuoft)
- [Hermann Grassmann - Wikipedia](https://en.wikipedia.org/wiki/Hermann_Grassmann)
- [pandas](https://pandas.pydata.org/) / [Polars](https://pola.rs/) — DataFrame の現代的実装
- Bender, Gebru, McMillan-Major, Mitchell (2021) ["On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? 🦜"](https://dl.acm.org/doi/10.1145/3442188.3445922) FAccT — form と meaning の区別
- 前回授業 [`lesson_04_thales_circumcenter.ipynb`](lesson_04_thales_circumcenter.ipynb) — §0 「嫌い → 抽象化」 pattern、§7 科学転生無双レポート
- 道具のロードマップ [`ggblab_extra_roadmap.md`](ggblab_extra_roadmap.md)
