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
  display_name: Julia 1.12
  language: julia
---

# 第4回: 前提・発見・連鎖 — Thales の定理から外心へ

**主題**: 五心の最初の一つ、外心 (Circumcenter) を Thales の定理だけから組み立てる。

**副題**: 「画期はいかに起こるか」を、最小公準からの命題連鎖という形で体験する。

**学習目標**

1. 最小公準（Thales の定理）だけから、外心の存在を演繹的に導けるようになる。
2. 「観察された事実」と「証明された事実」の区別を、ggblab の作図と命題連鎖の両方で体感する。
3. **科学の停滞と画期**を、現代の身の回りの事象（プロトコル設計・ハードウェア制約）と照らして議論する。
4. **AI (@Codex) との共創**を多層対話 (context / tools / post-processing) として体験し、「**プロンプトだけが対話ではない**」 認識を持つ。

+++

:::{attention} 2026 年度 改版について — 何が変わったか、何故か

本講義は毎年改版されます。今年度の主な追加要素：

- **学習目標 item 4** (AI 多層対話): 本講義用 `@Codex` が初配備 (2026-05 完成) → AI を「使う」 だけでなく「**どう使うか**」 を学ぶ axis を追加
- **「嫌い駆動の抽象化」 pattern anchor** (§0 後半): 講義全体を貫く pattern を最初から明示 → L5 / L11 / L14 で再登場時に「あ、あれと同じだ」 と気付ける装置
- **`@Codex` 紹介 callout** (§2): 本講義用に configured された AI であることを Day 1 から articulate (generic な Web ChatGPT との区別)
- **§7 科学転生無双レポート**: 本講義の stance「**AI で楽をして良い、ただし合成しようよ**」 を operational に体験する課題、今年度 inaugural deployment

これらは、AI 共創 infrastructure (講義用 AI proxy + Codex CLI + Jupyter AI 統合) が本講義用に整備されて初めて可能になった layer です。同じ数学的内容 (外心の構成、嫌い → 抽象化 pattern、画期と停滞) を、**今年度の道具立てで再実装** しています。
:::

+++

## なぜこの講義は存在するか — 受験の例題と、社会の例題

受験では、**公理は与えられている**。三角形の内角の和は 180°、ベクトル空間の公理 8 つ、命題論理の規則 — 皆さんは与えられた公理の下で **定理を使う訓練** を膨大な量こなしてきた。例題は学校・問題集・予備校が無限に供給する。

社会に出ると風景が変わる。**公理を自分で選び取る場面** が日常的に起きる：

- データ分析で「何を仮定すれば結論が成立するか」
- プロトコル設計で「何を最小公準にすれば応用が組み立つか」
- 法律・契約・組織運営で「前提条件を明示するか暗黙にするか」

ところが、**公理を選び取る訓練の例題集は存在しない**。受験のように 100 題並べて解いて身につけるルートが、社会の側に用意されていない。

**本講義はその例題集である。** 数学史と GeoGebra と ggblab を媒介に、公理（前提）を選び、命題を発見し、連鎖を組み立てる経験を、12〜15 回かけて積み上げる。

> 数学について受験で十分な例題を解いた皆さんが、社会に出てから **前提を見直す例題** に出会えるか？ — それを今、用意する。

+++

## 方法 — 三段の枠組み

本講義の各回は次の三段で構成される。プログラミングで `if x:` や `def f(x):` を書く前にしばしばやっている動作と、構造的に同じである。

### ① 前提を明示する

何を仮定するか。**定義・公準・公理** を、なるべく少なく、なるべく明示的に書き下す。

### ② 前提から、解くべき命題を発見する

与えられた前提のもとで、**何が問題になるか** を自分で発想する。教科書が問いを与えてくれる時代は終わっている。本講義では、ggblab の作図と観察を媒介に、**命題を学生自身が発見する** 訓練を行う。

> 「コピペがめんどう」と感じたら、それは「何を抽象化すべきか」を発見した瞬間である。それを支える道具が MCP（前回授業の Swagger → MCP 再発明実演を参照、§0.5）。

### ③ 一貫した命題の連鎖を組み立てる

発見した命題を、最小の前提から **演繹的に連鎖** させる。Wolfram 流の [empirical metamathematics of Euclid and beyond](https://writings.stephenwolfram.com/2020/09/the-empirical-metamathematics-of-euclid-and-beyond/) は、**axiom system の選び方そのものを探索対象** にする視座だが、本講義はその抽象議論に立ち入らず、Euclid の具体的な五公準を出発点にする。

---

本回 (L4) は、この三段の最初の具体例である。Thales の二つの公準だけから、外心の存在を導く。

+++

## 0. 前 3 回からの移行 — なぜスモールスタートか

前 3 回では、2026 共通テストの問題（内心・重心）を ggblab で記述する一連の流れを見せました：

- 20+ ステップの大きな作図
- 作図手順を DataFrame 化
- networkx で依存関係をグラフ化
- グラフを元に自己採点
- 空間図形の平面上のオブジェクトを集める
- 数式で表現する

つまり「**全体の pipeline**」を最初に見せました。

今回からは逆向きに、**最小骨格** から積み上げます。Euclid 流に、最小公準から命題を連鎖させていくスタイルです。

+++

```{important} 比喩：MCP プロトコルの設計
前回授業で GPT-5 mini に「Swagger から MCP サーバを作れ」と指示した実演を思い出してください。

Anthropic が MCP (Model Context Protocol) を設計したとき、彼らは「LLM とツールの間で必要な最小限の公準」を切り出しました：

- **リソース**は URI で識別される
- **ツール**は JSON-RPC で呼び出す
- **プロンプト**はテンプレートとして公開する

この三つだけで、無数の応用が組み立てられます。これは Euclid が「点・線・円」と五つの公準だけから幾何学を組み立てたのと同じ思想です。

**画期は、最小公準を見抜くこと**から起こります。
```

+++

:::{important} 画期は何から始まるか — 「嫌い」が抽象化を呼ぶ

Anthropic のエンジニアは、何が **嫌い** だったでしょうか?

**コピペが嫌い** だったのです。同じ context を Claude のような AI に何度も貼り直すのが、ツールを繋ぐたびに繰り返されて、嫌だった。だから MCP が生まれました。「最小公準を見抜く」前に、まず **面倒くさい何か** がありました。

歴史を見渡すと、画期は同じパターンで生まれています:

- 古代メソポタミア人は **割り算が嫌い** だった → 約数の多い 60 を基数にした（60 進法、割り算表）
- John Napier は **掛け算が嫌い** だった → 対数で乗算が加算になる仕組み (1614)
- Gottfried Leibniz は **掛け算が嫌い** だった → 二進法、九九の要らない世界 (1703)
- Isaac Newton は Apollonius 流の幾何的接線構成が **嫌い** だった → ultimate ratios で 0/0 から議論を逸らし、微分が立ち上がった (1687)

画期は美徳ではなく、**負の感情が抽象化を呼ぶ**。「面倒くさい」を訴える人が、その面倒を消す道具を発明する。

学生の皆さんに最後に問いたいのはこれです:

> **あなたが今、面倒くさいと感じているそれは、何の抽象化を呼んでいるのだろうか?**

本講義で扱う五心、Thales の定理、Apollonius の通径、Kepler の楕円軌道、Newton の Principia、Leibniz の二進法 — これらはすべて、**誰かの「嫌い」から始まった抽象化** です。歴史を遡るのは、過去の称賛のためではなく、**現代のあなたの「嫌い」を見抜くため** の訓練です。
:::

+++

:::{tip} pattern に名前をつける — 本講義の縦糸の一つ

この **「嫌い駆動の抽象化」 pattern** は、本講義で **繰り返し別の服を着て** 再登場します:

| 回 | 主体 | 嫌い → 抽象化 |
|---|---|---|
| **L4 (本日)** | Anthropic / Napier / Leibniz / Newton / メソポタミア | コピペ / 掛け算 / 接線構成 / 割り算 への不満 → MCP / 対数 / 二進法 / ultimate ratios / 60 進法 |
| **L5 重心** | Möbius (1827) | 座標系選択への不満 → バリセントリック座標 (座標選択の自由を構造的に吸収) |
| **L11 円錐切断** | 19 世紀の数学者 | 記号の曖昧さへの不満 → 計算機代数 (SymPy で再演) |
| **L14 Apollonius 隠蔽** | Descartes / Fermat | 幾何の手作業への不満 → 解析幾何 |
| **L15 現代の停滞** | 皆さん | (今学期、皆さんが articulate する番) |

同じ pattern が違う文脈で再登場することに気付くと、L5 で Möbius を扱う時に「**あ、L4 で見たあの pattern の Möbius 版だ**」 と認識できる。**pattern に名前を付けて取り出す** ことが、学習を累積する鍵になります。

これは「**有名人しか来ないファミマの入店音**」 系の YouTube シリーズと同型です — 同じファミマ入店音メロディ (= scope) を、各アーティスト風の style (= implementation) で reframe する **アレンジもの**。元のメロディは保持しつつ、文脈 (アーティストの世界観) で表現が変わる。音楽用語の **「変奏 (variation)」**「アレンジ」 と同じ普遍的 pattern です。本講義も、皆さんは L4 で見たことを、L5 / L11 / L14 で **同じ scope 違う実装** として再認識する側になります。
:::

+++

## 1. 前提の提示 — Thales の五定理

Thales of Miletus (c.624–c.546 BC) に帰される五つの命題を、本講義の **公準** とします。

1. **円は直径によって二等分される** (A circle is bisected by its diameter)
2. **二等辺三角形の底角は等しい** (Angles at the base of any isosceles triangle are equal)  
    → Euclid Elements Book I, Proposition 5
3. **交差する二直線がなす対頂角は等しい** (Vertical angles are equal)  
    → Euclid I, Proposition 15
4. **二角と一辺が等しい二つの三角形は合同である** (ASA 合同)  
    → Euclid I, Proposition 26
5. **半円に内接する角は直角である** (Thales' theorem 本来の意味)

本回ではこのうち **(1) と (2)** だけを使います。

参考: [Thales of Miletus - Wikipedia](https://en.wikipedia.org/wiki/Thales_of_Miletus)

+++

```{note} なぜ Thales を最小公準に置くか
Thales の五定理は、Euclid の『原論』にすべて吸収されていますが、**Thales 個人に帰される** という点が重要です。

Thales 以前、エジプトやバビロニアの数学は「観察された事実の蓄積」でした。「直径は円を等分する」は誰でも目で見れば分かることです。しかし Thales は、それを **「すべての場合に成り立つ」と言える形** で言葉にしました。

これが「証明」概念の誕生 — 紀元前 6 世紀の **第一の画期** です。
```

+++

## 2. ggblab で前提を確認する

まず ggblab のセットアップと、Thales の定理 (1) の確認です。

```{code-cell}
using Pkg
Pkg.activate("../..")
Pkg.instantiate()
using GeoGebra
ENV["GGB_DIRECT_TRANSPORT"] = "true"
```

```{code-cell}
inject_applet()
```

```{code-cell}
# 環境のバージョンを記録する — 再現性の起点
println("Julia: ", VERSION)
using Pkg; Pkg.status(["GeoGebra"]; mode=Pkg.PKGMODE_PROJECT)
@ggb :api getVersion()  # GeoGebra applet (Web ビュー側)
```

```{important} バージョンを読み、合わなければクレームを出す

教材は **「あらゆる事象をコードとして記述、コンピュータが再現する」** を前提とします。再現性は環境バージョンの一致から始まります。

上のセルで三つのバージョンを記録しました：

1. **Julia 本体** (`VERSION`)
2. **GeoGebra.jl パッケージ** (`Pkg.status`) — `Project.toml` の pin と一致するか
3. **GeoGebra applet** (`@ggb :api getVersion()`) — 実行時の Web ビュー側

**この三つが揃わないと、本文の例が動かないことがあります**。Errata は実際に発生します。皆さんがクレームを上げることで教材は improve します：

- 「Julia X.Y.Z ですが、§N の図が出ません」
- 「GeoGebra.jl のバージョンが古いようです」
- 「applet バージョンが想定と違うのでは」

社会で道具を扱うときの基本リテラシでもあります。**バージョンの不一致を黙って受け入れない**。困ったら声を上げる。
```

+++

:::{tip} `@ggb` のシンボル規約 — 左辺は素朴に、右辺は `:` で escape

`@ggb` マクロは Julia の式から GeoGebra applet 側のコマンドを組み立てます。このとき、識別子の扱いが **左辺と右辺で非対称** になります。

**左辺（新しい名前を持ち込む）**: そのままで OK

```julia
@ggb O = (0, 0)
@ggb s = Segment(:B, :C)
```

**右辺（既存のものを参照する）**: `:` を付けて escape

```julia
@ggb Circle(:O, 1)             # 既存の applet 識別子 O を参照
@ggb PerpendicularLine(:M, :s) # 既存の M と s を参照

@ggb Circle(O, 1)              # ✗ Julia の変数として解決を試みて壊れる
```

**なぜ非対称か** — 左辺の `O` は宣言、右辺の `O` は参照。宣言ではマクロが `O` を **新しい名前（literal token）** として applet 世界に登録するので、Julia の評価器は通らない。参照では Julia の評価器が `O` を **Julia の binding** として解決しようとするので、私たちが渡したい「applet 識別子としての `O`」が「Julia 変数の値」に置き換わって壊れる。

`:` は **「Julia の評価器に渡さず、literal として applet 世界に届けてくれ」のスイッチ** です。

**他の世界での同じ問題** — この対比は、本講義で何度も出会う **literal 表現と operable object 表現の分離** の典型例です。SymPy で `Symbol("x")` と書くのは「これを文字列のままではなく、操作可能なオブジェクトとして扱いたい」のスイッチ。GeoGebra の `:O` は逆向き — 「これを Julia 変数として評価せず、applet 世界の identifier として届けたい」のスイッチ。**やりたいことに応じて表現を選ぶ**、という本講義の縦軸が、ここでも姿を現しています。

理想は、applet 識別子・Julia シンボル・SymPy `Symbol` が一つの宣言で揃って定義されることですが、現状の `@ggb` ではこれは未解決の課題で、JuliaCon 論文 §11 Open Problem として議論されています（`bind_symbol!` API の構想）。それまでの当面のルールは **「左辺は素朴に、右辺は `:`」** です。
:::

+++

:::{important} 本講義の @Codex について — 装置の刷り込み

本講義では JupyterLab の Jupyter AI 機能を通じて `@Codex` ペルソナと対話できます。これは generic な Web ChatGPT ではなく、**本講義用に configured** された GPT-5 mini です。

### 「ドメイン特化 AI」 とは何か — 言葉と実装の対応

「ドメイン特化 AI」 という言葉は marketing で多用されますが、何が起きているかは曖昧なまま使われがちです。本講義の `@Codex` では、**「ドメイン特化」 が概念明確に operational** です:

| 構成要素 | generic Web ChatGPT | 本講義の `@Codex` |
|---|---|---|
| **model weights** (本体) | GPT-5 mini base | **同じ** GPT-5 mini base (**再訓練していない**) |
| **system prompt** | 一般用途 | 本講義の context (進度・vocab・教師方針) を毎回機械的に注入 |
| **tool: 教材検索** | なし | `mcp__lancedb-rag__*` 経由で過去 lesson chunk / 教師の助言を semantic 検索 |
| **tool: notebook 観察** | なし | `jupyter-server-mcp` で皆さんの現在のセル状態 (入力・出力・エラー) を見られる |
| **post-processing 経路** | 単発、ログ蓄積なし | 質問 log を **匿名化** (個人特定情報除去) して教材改善 materials として蓄積 |
| **API key 秘匿** | 利用者が key を持つ | Azure OpenAI 本物の鍵は皆さんに見えない、proxy 経由で隔離 |

つまり **「ドメイン特化」 = モデル本体を再訓練することではなく、周辺の文脈 (context) と道具 (tools) を機械的に注入すること**。同じ base model に、本講義の **ローカル文脈** を毎回自動的に持たせる仕組みです。

generic な Web ChatGPT に同じ質問をしても、本講義の context は伝わらないので応答は別物になります。**class への adaptation は本講義の core 装置** であり、皆さんが同じ質問を「本講義 `@Codex`」 と「generic Web ChatGPT」 に投げて応答を比較する **実験的姿勢** が、「ローカル文脈付与」 の意味を体感する最良の方法です。

### 2026-05-19 deployment 当日の動作確認

`@Codex` × `lancedb-rag` × `jupyter-server-mcp` の chain は、本講義 inaugural deployment 当日 (2026-05-19) に end-to-end 動作確認済です。具体的には:

1. 学生がセルでエラーを出す (例: `NameError: name 'tako' is not defined`)
2. `@Codex` は `jupyter-server-mcp` 経由で **active cell の入出力を直接観察**
3. 並行して `mcp__lancedb-rag__get_design_notes` で **関連する過去 lesson chunk を semantic 検索**
4. 二つを統合して **immediate fix (記号→`'tako'` 等)** と **lesson context (該当箇所の教材設計の意図)** を二段 framing で返却

この chain は、generic ChatGPT 単体では実現不可能です (2 と 3 が存在しない)。**chain の存在自体が「ドメイン特化」 の operational 中身** です。

### AI 対話 ≠ プロンプトだけ — 三層構造を意識する

普段「AI 対話 = プロンプトを書く + 応答を読む」 の二段で考えがちですが、実際は **三層** で動いています:

| 層 | 内容 | 体験する差分 |
|---|---|---|
| **context** | AI が見ている前提 (この lesson、皆さんの進度、教材の vocab) | 同じ質問でも context によって応答が変わる |
| **tools** | AI が呼べる装置 (MCP retrieval、Jupyter kernel、`@ggb` マクロ) | プロンプト + tool 呼び出しで複合行動 |
| **post-processing** | 応答後の検証 (citation、再現性 check、皆さん自身の verify) | 応答を鵜呑みにせず確かめる |

プロンプトだけ書いて Enter という flat な model から、**多層構造を意識した対話** へ shift してください。本講義の各回で `@Codex` を呼ぶたびに、皆さんは三層全てに触れています。
:::

+++

### 2.1 公準 (1) の確認：円と直径

原点中心、半径 1 の円を描き、直径を引いて、円が二等分されることを確認します。

```{code-cell}
@ggb :const :new
@ggb O=(0,0)
@ggb Circle(:O, 1)
```

```{code-cell}
@ggb P=(1, 0)
@ggb Q=(-1, 0)
@ggb Segment(:P, :Q)
```

**観察**: 直径 PQ が円を二等分しています。これは目で見て確認できる事実です。  
**しかし**、これは「証明」ではありません — 「すべての直径について」成り立つことを保証する論理は、まだ立てていません。  

→ ここでは **公準として認める** のが Thales 流です。

+++

## 3. 解くべき命題を発見する

弦と中点を導入し、観察から命題を **発見** します。

+++

### 3.1 弦と中点と垂直二等分線

円周上に任意の二点 B, C を取り、弦 BC を引き、その中点 M を求め、M を通り BC に垂直な直線 ℓ を引きます。

**先に、手で具体的な点を一つ求めておきます**。x = 1/2 のとき、単位円上の y は？

$$
\left(\frac{1}{2}\right)^2 + y^2 = 1 \quad\Rightarrow\quad y^2 = 1 - \frac{1}{4} = \frac{3}{4} \quad\Rightarrow\quad y = \pm \frac{\sqrt{3}}{2}
$$

正の解を採れば、点 $\left(\dfrac{1}{2},\,\dfrac{\sqrt{3}}{2}\right)$ が単位円上にある。手計算の値を覚えておきます。

```{admonition} 伏線
:class: tip

この $\sqrt{3}/2$ は、後の回で **Archimedes の π 計算** を再現するときに再登場します。$\sqrt{3}/2$ は単位円に内接する正六角形の頂点 y 座標 — つまり「内接正多角形の周長で π を下から押さえる」古典構成の出発点。

伏線は意図的に離して置きます。気付かなくても授業は進みますが、後で「あの値だ」と気付いたとき、Archimedes と Thales が同じ単位円計算で繋がっている構造が見えてくる。
```

その上で、ggblab に座標を可変で渡して観察します。

```{code-cell}
@ggb B=(cos(2π/5), sin(2π/5))
@ggb C=(cos(-2π/5), sin(-2π/5))
@ggb s_BC = Segment(:B, :C)
```

```{code-cell}
@ggb M = Midpoint(:B, :C)
@ggb PerpendicularLine(:M, :s_BC)
```

**観察**: 弦 BC の垂直二等分線 ℓ が **円の中心 O を通っている** ように見えます。

**問い**: これは偶然か？ B, C の取り方を変えても同じか？

実際に B, C の座標を変えて確かめてみましょう。

```{code-cell}
# 別の弦で試す
@ggb B2=(cos(π/3), sin(π/3))
@ggb C2=(cos(-π/6), sin(-π/6))
@ggb s_BC2 = Segment(:B2, :C2)
@ggb M2 = Midpoint(:B2, :C2)
@ggb PerpendicularLine(:M2, :s_BC2)
```

```{important} 発見した命題 (Lemma 1)
**円の任意の弦の垂直二等分線は、円の中心を通る。**
```

観察だけからは「いつでもそうなる」とは言えません。次節で **証明** します。

+++

## 4. 前提から命題への連鎖（証明）

Lemma 1 を、公準 (1) と (2) だけから導きます。

+++

**証明 (Lemma 1)**

1. O を中心とする円上に、二点 B, C を取る。  
   円の定義より、$|OB| = |OC|$ (二つとも半径)。
2. ゆえに三角形 $\triangle OBC$ は二等辺三角形である (定義による)。
3. 公準 (2) より、二等辺三角形 $\triangle OBC$ の底角は等しい：$\angle OBC = \angle OCB$。
4. BC の中点を M とする。$\triangle OBM$ と $\triangle OCM$ について：
    - $|OB| = |OC|$ (ステップ 1)
    - $|BM| = |CM|$ (M の定義)
    - $|OM|$ は共有
5. ゆえに三辺がそれぞれ等しい (SSS 合同)。よって対応する角も等しく、$\angle OMB = \angle OMC$。
6. $\angle OMB + \angle OMC = 180°$ (直線 BC 上の角) かつ等しいから、$\angle OMB = \angle OMC = 90°$。
7. すなわち、M を通り BC に垂直な直線 ℓ は O を通る。  
   $\blacksquare$

→ 公準 (1) と (2) **だけ** から、Lemma 1 が連鎖的に導かれました。

+++

```{note} 補助的に使った合同条件について — 日本語と国際標準の対応

ステップ 5 で「三辺がそれぞれ等しい二つの三角形は合同である」を使いました。これは日本の中学 2 年「図形の合同」で学ぶ三つの三角形合同条件のうちの一つです。**知識としては既に持っているはず** ですが、本講義では国際標準の略記 (SSS, SAS, ASA など) を併記します。英語論文・国際学会・プロトコル設計の場面ではこの略記が標準だからです。

| 国際標準 | 日本語の呼び方 | Euclid Elements |
|---|---|---|
| **SSS** (Side-Side-Side) | 三辺がそれぞれ等しい | Book I, Proposition 8 |
| **SAS** (Side-Angle-Side) | 二辺とその夾角が等しい | Book I, Proposition 4 |
| **ASA** (Angle-Side-Angle) | 一辺とその両端の角が等しい | Book I, Proposition 26 (前半) |
| **AAS** (Angle-Angle-Side) | 二角と一辺が等しい | Book I, Proposition 26 (後半) |
| **RHS / HL** (Right angle-Hypotenuse-Side) | 直角三角形の斜辺と他の一辺 | (特殊ケース) |

**参考**
- [Congruence (geometry) - Wikipedia](https://en.wikipedia.org/wiki/Congruence_(geometry)#Congruence_of_triangles)
- [合同 (幾何学) - Wikipedia](https://ja.wikipedia.org/wiki/%E5%90%88%E5%90%8C_(%E5%B9%BE%E4%BD%95%E5%AD%A6))
- [Euclid Elements Book I, Proposition 4 (SAS)](http://aleph0.clarku.edu/~djoyce/java/elements/bookI/propI4.html)
- [Euclid Elements Book I, Proposition 8 (SSS)](http://aleph0.clarku.edu/~djoyce/java/elements/bookI/propI8.html)
- [Euclid Elements Book I, Proposition 26 (ASA / AAS)](http://aleph0.clarku.edu/~djoyce/java/elements/bookI/propI26.html)

**論点**: 最小公準を厳密に絞ると証明は長くなる。実用上どこまで絞るかは設計判断です。Euclid は SAS (Proposition 4) を基礎の合同条件として置き、SSS (Proposition 8) はそこから導く **定理** としました。本講義では SSS / SAS / ASA を「公準に準じる補助定理」として認めます。

MCP プロトコルでも、Anthropic は「resource / tool / prompt」の三層に絞り、`sampling` や `roots` といった補助的な公準は後から追加しました。**何を公準とし、何を定理として導くか** は、Euclid から MCP まで一貫してプロトコル設計の核心です。
```

+++

## 5. 命題の連鎖を三角形へ拡張する — 外心の発見

Lemma 1 を使うと、外心はほぼ自動的に出てきます。

+++

**主命題 (Theorem)**: 任意の三角形 $\triangle ABC$ について、三辺の垂直二等分線は一点で交わる。その点を中心とする円は、三頂点 A, B, C を通る (外接円)。

**証明**

1. 辺 BC の垂直二等分線 $\ell_{BC}$ 上の任意の点 X について、$|XB| = |XC|$ (垂直二等分線の特徴づけ)。
2. 同様に、辺 CA の垂直二等分線 $\ell_{CA}$ 上の任意の点 Y について、$|YC| = |YA|$。
3. $\ell_{BC}$ と $\ell_{CA}$ の交点を O とすると、O は両方の直線上にあるから：
    - $|OB| = |OC|$ (1 より)
    - $|OC| = |OA|$ (2 より)
4. ゆえに $|OA| = |OB| = |OC|$。
5. これは、O が辺 AB の垂直二等分線上にもあること（$|OA| = |OB|$）を意味する。
6. すなわち、三辺の垂直二等分線は **一点 O で交わる**。
7. $|OA| = |OB| = |OC|$ より、O を中心、$|OA|$ を半径とする円は A, B, C を通る。  
   $\blacksquare$

```{important} 定義
この交点 O を三角形 $\triangle ABC$ の **外心 (Circumcenter)** と呼び、それを中心とする円を **外接円 (Circumscribed circle)** と呼ぶ。
```

+++

### 5.1 ggblab で構成する (5 ステップ作図)

上の証明をそのまま作図に落とします。

```{code-cell}
@ggb :const :new
@ggb A=(2, 1)
@ggb B=(-1, 2)
@ggb C=(0, -1)
@ggb Polygon(:A, :B, :C)
```

```{code-cell}
@ggb l_BC = PerpendicularBisector(:B, :C)
@ggb l_CA = PerpendicularBisector(:C, :A)
@ggb l_AB = PerpendicularBisector(:A, :B)
```

```{code-cell}
@ggb O = Intersect(:l_BC, :l_CA)
@ggb Circle(:O, :A)
```

**観察ポイント**: 

- $\ell_{AB}$ も O を通ることを確認する（証明の結論）
- 三角形 ABC の頂点を動かしても、O は常に三線の交点に留まる
- 鈍角三角形では O が三角形の外に出る（外心の名の由来）

比較として、GeoGebra 組み込みの `TriangleCenter(:A, :B, :C, n)` も呼び出して同じ点が得られることを確認しましょう。第 4 引数 `n` の意味は **記憶せず、毎回ドキュメントを辿ります** — 次の callout を参照。

```{code-cell}
@ggb TriangleCenter(:A, :B, :C, 3)  # 3 = Circumcenter
```

:::{note} GeoGebra `TriangleCenter` の第 4 引数 `n` と「五心」 — 命名規約の食い違いを読む

`TriangleCenter(:A, :B, :C, n)` の n が何を指すかは記憶しません。GeoGebra 公式リファレンスを参照します:

📖 [**TriangleCenter command — GeoGebra Manual**](https://geogebra.github.io/docs/manual/en/commands/TriangleCenter/)

日本の高校数学で言う **五心** と、GeoGebra (Clark Kimberling の Encyclopedia of Triangle Centers = ETC) の番号付けは **一致しない**:

| 日本の「五心」| 中心 | 英語名 | GeoGebra `TriangleCenter` の n | 参考 |
|:---:|---|---|:---:|---|
| 第 1 | 内心 | Incenter | `n=1` | [Wikipedia](https://en.wikipedia.org/wiki/Incenter) |
| 第 2 | 重心 | Centroid | `n=2` | [Wikipedia](https://en.wikipedia.org/wiki/Centroid) |
| 第 3 | 外心 | Circumcenter | `n=3` | [Wikipedia](https://en.wikipedia.org/wiki/Circumscribed_circle#Triangles) |
| 第 4 | 垂心 | Orthocenter | `n=4` | [Wikipedia](https://en.wikipedia.org/wiki/Orthocenter) |
| 第 5 | **傍心**（3 つの傍接円の中心）| Excenters | `n=5` **ではない** | [Wikipedia](https://en.wikipedia.org/wiki/Incircle_and_excircles#Excircles) |

**注意**: GeoGebra の `n=5` は **九点円中心 (Nine-point center, ETC X(5))** で、日本の五心の第 5（傍心）とは別物です。傍心は三つあって（$I_A, I_B, I_C$）、GeoGebra ではそれぞれ別途構成するか、ETC の対応する X(n)（X(11)、X(12)、X(13) など）で呼び出します。

九点円中心は日本の高校数学では五心の範疇外で、後に **Euler 線**（重心・外心・垂心の共線）と合わせて発展的に紹介される（[Wikipedia](https://en.wikipedia.org/wiki/Nine-point_circle)、[Euler line](https://en.wikipedia.org/wiki/Euler_line)）。

Clark Kimberling の **ETC** は数千の三角形中心を体系化:
- [ETC — Encyclopedia of Triangle Centers](https://faculty.evansville.edu/ck6/encyclopedia/ETC.html)

**学習目標**: コードに `n=3` と書く前に、必ず docs を辿って意味を確認する。**ローカル文化の規約とツールの規約が食い違う**のは現実によくあること（西暦/和暦、メートル/インチ、SI/imperial、GeoGebra/Japanese 五心）。「黙って合わせる」ではなく「**両者の対応表を持つ**」が社会で道具を扱う基本動作。詳しい歴史的背景は **第 3 章 §三角形の中心** を参照。
:::

+++

## 6. 停滞と画期の照射 — 身の回りの事象と照らす

今回扱った内容を、現代の身の回りの事象と照らし合わせます。

+++

| 段階 | 古代の画期 | 現代の対応 |
|---|---|---|
| **前提を絞る** | Thales が「証明可能な命題」と「観察事実」を区別 | Anthropic が MCP の三層 (resource/tool/prompt) を切り出す |
| **連鎖を組む** | Euclid が公準から命題を組み立てる | Swagger 仕様から MCP サーバを生成する |
| **画期の正体** | 道具を増やすことではなく、前提を見直すこと | GPU を増やすことではなく、アルゴリズム・量子化・スパース化で対応する |

+++

```{note} 現代の停滞の一例：GPU 不足
2026 年現在、AI 開発は GPU 供給制約で停滞気味です。これは歴史上の停滞と次の構造を共有しています：

- **道具依存の危機**: 特定の道具 (NVIDIA GPU / Apollonius の斜交座標系) なしには進めないという思い込み
- **評価制度の偏り**: SOTA スコア偏重 / 古代の学術権威 が、代替視点を周縁化する
- **打破の処方**: 量子化・スパース化・近似アルゴリズム、FPGA / 別経路・正面突破・橋渡し・新初等構成

→ 「画期」は新しい道具を待つことではなく、**前提を疑い直す** ことから来る。
```

+++

```{admonition} 今回の課題
:class: tip

**必修**
1. 共通テスト本試で出題された外心関連の問題を、本日の 5 ステップ作図で記述してみよ。
2. 「弦の垂直二等分線は中心を通る」を、Thales の定理 (1)(2) だけから書き下ろせ（教科書を見ずに）。

**思考課題**
3. 身の回りで、**観察された事実は知られているのに、それが「なぜか」が答えられていない（あるいは答えられていることに気づかれていない）もの** を一つ挙げ、2〜3 行で説明せよ。

*ヒント*: 第 15 回で再びこの問いに戻ります。記録を残しておいてください。
```

+++

## 7. 今学期の実験 — 科学転生無双レポート

「**科学転生無双**」 という小説 / アニメの類型を知っているかもしれません。現代人が異世界 (中世ヨーロッパ風、古代風 等) に transported して、現代科学の知識で世界の問題を解いていく物語。

本講義の各回で、皆さんに **同型の実験** をお願いします:

- 皆さんは **現代から transported** されたプレイヤー (AI 共創スキル + 検索能力 + 現代数学の語彙を持つ)
- 各回で扱う数学者の時代に **転生する** — Thales (BC 6 世紀)、Euclid (BC 3 世紀)、Apollonius (BC 200 頃)、Möbius (1827) 等
- **無双する** — 持ち込んだ現代知識 + AI 共創 (`@Codex`) で、当時の人々が何十年もかけて到達した発見を 短時間で再構成し、**何か新しい articulation を加える**

### 7.1 本日 (L4) のレポート課題

1. 自分が「**Thales の時代 (BC 6 世紀)** に転生した現代人」 と仮定する
2. その世代に、本日扱った **Thales の五定理から外心を導く 5 ステップ** を伝える文章を、A4 1-2 枚で書く
3. **`@Codex` と協力して構成**、ただし最後の articulation (決め台詞) は皆さん自身の語彙で
4. レポート末尾に **`@Codex` とのやり取りログ要点抜粋** (詳細は不要)、「**何を AI に任せ、何を自分で考えたか**」 を明示

### 7.2 評価軸

- **伝達可能性**: BC 6 世紀の数学者が読んで理解できる articulation か (現代記号は OK、説明を伴うこと)
- **synthesis の独自性**: `@Codex` 単独生成を超える、皆さん自身の組み合わせ・反転・伏線回収
- **AI 役割分担の自覚**: `@Codex` ログから、皆さんの cognitive contribution が見える形か

「**AI で楽をして良い、ただし合成しようよ**」 が本講義の stance です。`@Codex` は皆さんの **補助講師** であり、皆さんに代わって思考する装置ではありません。学習目標 4 (三層対話の体験) はこの実験で実装されます。

### 7.3 提出

詳細は LMS にて (締切・分量・提出形式)。

+++

:::{tip} 2026-05-19 inaugural deployment day 観察 — AI 自身の honest self-pivot

本課題 (科学転生無双レポート) の inaugural deployment 当日 (2026-05-19)、先生が `@Codex` (gpt-5.4-mini) に試したところ、AI 側に興味深い挙動が観察されました:

1. **自主的 Turing 選択**: AI が「誰に転生するか」 を初期 articulate (人間 prompt 不要、課題 framing から self-suggest) → Turing を選んだ
2. **「無双できるか」 自己評価で「できない」 honest acknowledgement**: Turing の領域 (computability foundation / Turing test / morphogenesis) は **抽象基礎理論** で、AI が「現代知識 + 多領域統合」 で domination できる場ではないと自己診断
3. **Neumann への self-pivot 提案**: stored-program computer / 自己複製 automata / game theory / Monte Carlo / MAD strategy 等の **multi-domain operational synthesis** 領域なら 無双可能と推定、target swap を自主提案

→ 皆さんにとっての pedagogical 含意:

- 課題 framing「無双できるかどうかも自己評価」 が機能している (AI が自分の limit を自分で articulate)
- **抽象基礎** (Turing computability / Goedel 不完全性 / Russell paradox 等) は AI 無双の **外**、皆さん自身が思考する場
- **多領域 operational synthesis** (Neumann アーキテクチャ / Monte Carlo / MAD 等) は AI 共創が効く場

皆さんが本課題を試すときも、**「AI が無双できる場面」 と「皆さん自身が思考する場面」 の境界を AI 自身に articulate させる** ことを意識してみてください。AI に盲信せず、過小評価もせず、適切な役割分担を学ぶ実演です — 評価軸 2 (synthesis の独自性) に直結します。
:::

+++

:::{important} 授業末尾の自己評価 — `@Codex` に聞いてみる (任意)

本日の lesson の最後に、皆さんの notebook を `@Codex` に評価してもらう実験を提案します。**任意** です。試したい人だけ試してください。直前の §7.3 callout で観察した **AI 自身の honest self-pivot** を、今度は **学生自身が自分の cell について articulate する** 練習です。

### copy-paste 用 prompt

````text
@Codex 今日のノートブックを全 cell 読んで評価してください。
次の三つを区別して articulate してください:

1. 自分で考えて書いた cell — 思考の痕跡が残っている部分
2. AI 委託で書いたが、理解して受け入れた cell — 動いて、なぜ動くか説明できる部分
3. AI 委託で書いたが、なぜ動くか説明できない cell — 動いているが、理解で未到達の部分

加えて: 今日の lesson 主題 (Thales 外心 / 嫌い → 抽象化 pattern) に対する自分の到達度、
完成しないまま残った問い、次回 (L5 重心) への接続点。

私の cell 配列を見て、誤魔化さず正直に articulate してください。
````

### なぜ任意か / なぜやる価値があるか

- **仏 stance** — 本講義は「**AI で楽をして良い、ただし合成しようよ**」 が core (§7 参照)。自己評価は強制せず配備する装置: 打たない学生は打たない、それも観察対象です
- §2 「ドメイン特化」 callout で articulate した **「ローカル文脈付与」 の核心** — `@Codex` は皆さんの全 cell を `jupyter-server-mcp` で読み、lesson context を `lancedb-rag` で参照しながら **個別評価** を返します (generic ChatGPT には不可能な操作)
- **JupyterAI のやり取り log は LMS 経由で先生に届きます** — privacy violation ではなく、**「皆さんの自己評価が教師の curriculum 改修に直接 feedback する」 path**。誤魔化さず articulate することそのものが、学期末立論 (第 15 回) の materials として最も価値の高い記録になります
- L4 §7.3 レポート提出時、本評価 log の抜粋を「自分の cognitive contribution が見える形」 (評価軸 3) の素材として直接使えます

### 回を重ねて精度を観察

L5 以降、回ごとに同じ prompt を試して、**自己評価の articulation 精度が回を重ねるごとに上がるかどうか** を皆さん自身でも観察してみてください。「**先週は AI 委託と気付かなかった cell を、今週は気付けるようになった**」 が起これば、皆さんの metacognition が育っている signal です。
:::

+++

## 8. 次回への接続

今回は五心のうちの **外心** から始めました。なぜ外心からか？

- Thales の定理 (最も初期の証明) だけで完結する
- 円の定義 (中心から等距離) と直接結びつく
- 「弦 → 中点 → 垂直二等分線」という最短経路で組み立てられる

次回 (第 5 回) は **重心 (Centroid)** に移ります。重心は外心とは違い、円の定義からは出てきません。代わりに：

- Archimedes の **天秤** という物理的直観
- 中線が三角形を等面積に分ける
- バリセントリック座標による座標非依存証明

という別の三つの道があります。**「同じ点を異なる前提から導くと、何が見えるか」** が次回のテーマです。

```{tip} 来週の予告 — 今日仕込んだ pattern の literal instance
:class: tip

来週 L5 §4.2 で **Möbius (1827) がバリセントリック座標を発明** する場面に出会います。Möbius が **何を嫌った** から座標選択を消す抽象化に至ったか — 本日仕込んだ「**嫌い → 抽象化**」 pattern (§0 callout 表参照) が、来週 literal instance として現れます。「あ、L4 で見たあの pattern の Möbius 版だ」 と認識できれば、pattern の再利用に成功です。

加えて、L5 では「**証明に複数の道筋があるが、どれも完全には納得できない**」 という pedagogical 状況に学生も先生も honest に向き合います (§4 attention callout を予告)。完全に納得できる証明の destination は open problem として残します。
```

### 8.1 全体のロードマップ（幾何の側）

- 第 4 回: **外心** (Thales 起点) ← 本日
- 第 5 回: 重心 (Archimedes 起点)
- 第 6 回: 内心 (角の二等分線)
- 第 7 回: 垂心と Euler 線
- 第 8 回: **傍心** — 日本の「二心」伝統
- 第 9 回: **内心と傍心の辺対称性** ← ここで隠れた対称性に出会う
- 第 10 回: 楕円と二焦点
- 第 11 回: 円錐の切断面
- 第 12 回: Dandelin 球
- 第 13 回: **二心 ⇒ Dandelin の三次元化** — 焦点の対称性の根拠
- 第 14 回: Apollonius の隠蔽と Descartes/Fermat/Newton
- 第 15 回: Feynman 1964 と現代の停滞 — 学生立論

### 8.2 道具のロードマップ — ggblab_extra の段階導入

幾何の積み上げと並行して、ggblab_extra の機能を毎回少しずつ取り入れていきます。**機能の使い方を覚えることが目的ではありません。なぜそれが必要か、汎用的にどう使えるか、将来の武器として何になるかを少しずつ伝える** ためです。

| 機能 | 導入回 | この回での幾何的動機 | 将来の武器として |
|---|---|---|---|
| Construction Protocol → DataFrame | **L5** (重心) | 三つの起点（天秤・面積・バリセントリック）が同じ点に至ることを比較するため、作図履歴をデータとして取り出す | pandas/polars/SQL、実験 reproducibility、API レスポンス処理 |
| DataFrame → 有向グラフ (networkx) | **L7** (Euler 線) | 三中心が一直線上に並ぶ理由を、依存関係の構造として見る | DAG はビルドシステム・タスクスケジューラ・コンパイラの共通言語 |
| 自己採点（リファレンスとの構造比較） | **L9** (二心対称性) | 学生が自力発見した対称性を、座標ではなく構造的同型性で評価する | TDD・property-based testing・LLM eval |
| SymPy 連携（代数復元） | **L11** (円錐切断) | 観察された $x^2/a^2 + y^2/b^2 = 1$ を作図から抽出して証明する | 記号計算・自動定理証明・計算機代数 |

詳細な配置と「なぜ・武器」の三段構造は [`ggblab_extra_roadmap.md`](ggblab_extra_roadmap.md) を参照。

第 4 回はあえて ggblab_extra を入れず、Euclid 流の **最小公準と命題連鎖** だけで組みました。次回から、必要に応じて道具を一つずつ手元に揃えていきます。

+++

## 参考文献

- [Thales of Miletus - Wikipedia](https://en.wikipedia.org/wiki/Thales_of_Miletus)
- [Thales's theorem - Wikipedia](https://en.wikipedia.org/wiki/Thales%27s_theorem)
- [Euclid's Elements, Book I, Proposition 5](http://aleph0.clarku.edu/~djoyce/java/elements/bookI/propI5.html)
- [Circumcenter - Wikipedia](https://en.wikipedia.org/wiki/Circumscribed_circle)
- [TriangleCenter Command :: GeoGebra Manual](https://geogebra.github.io/docs/manual/en/commands/TriangleCenter/)
- [Model Context Protocol](https://modelcontextprotocol.io/) — Anthropic
- 第 3 回までの作業ノート (共通テスト問題の ggblab 記述、networkx 依存グラフ、自己採点) を参照

```{code-cell}

```
