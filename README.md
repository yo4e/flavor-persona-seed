# Flavor Persona Seed

**Personality can emerge from choices, not descriptions.**  
人格を形容詞で定義する代わりに、「その人が何を選ぶか」を人格のシードとして使えないかを試す思考実験です。

## What is this?

`Flavor Persona Seed` は、AIペルソナやゲームNPC、創作キャラクターなどに対して、

- 明るい
- 誠実
- 内向的
- 冒険好き

といった抽象的な性格ラベルを大量に与える代わりに、**意味を持つ具体的な選択肢を順位づけして与える**ことで、人格の方向性や微妙な個体差を生み出せるかを検証するプロジェクトです。

最小の例は次のようなものです。

```text
favorite #1: A
favorite #2: B
favorite #3: C
avoid: D
```

A〜Dは単なるIDではなく、それぞれ味・色・質感・文化的連想などを持つ具体的な選択肢です。

同じ基本ペルソナでも、この選好の並びが変わるだけで、モデルが補間する人物像や語り口、価値判断、美意識に差が生まれるのではないか、というのが中心仮説です。

---

## Origin — “31 Flavors Test”

この発想は、AIとの雑談中に「もしAIに好きなアイスクリームのフレーバーを聞いたら、その答えは人格らしさを反映するのではないか？」という話から生まれました。

たとえば、同じ「19歳・地雷系ギャル」という設定でも、

```text
1st: 渋い和風フレーバー
2nd: 派手なキャンディ系
3rd: ラム系
```

という好みなら、

- 見た目とは違って味覚は渋い？
- 根は素朴？
- 人に合わせず好きなものを選ぶ？
- 家族や幼少期の記憶が関係している？

といった、明示していない人格の奥行きをモデルが自然に補完する可能性があります。

逆に、

```text
1st: 派手なキャンディ系
2nd: ベリー系
3rd: チョコ系
```

なら、同じ基本属性でも別の人物像が立ち上がるでしょう。

この雑談上の思考実験を、便宜的に **“31 Flavors Test”** と呼びました。

ただし、このリポジトリは特定のアイスクリームブランド、商品、商標を再現・利用することを目的としていません。実験や実装では、独自に設計した架空の選択肢セットや、別カテゴリの選択肢を使うことを前提とします。

---

## Core hypothesis

### 1. 抽象的な形容詞より、具体的な選択のほうが人格を立体化することがある

```text
calm, intelligent, friendly
```

だけを与えるより、

```text
favorite flavor: roasted tea + red bean
usual drink: black coffee
hotel preference: old small inn
phone case: changes every few months
```

のような具体的な選択を与えたほうが、モデルにとって「この人物なら次に何を選ぶか」の補間材料が増えます。

### 2. 選択肢は、人格の“座標”として使える

意味のある選択肢集合の中から、順位付きでいくつかを選ぶ。

31個の候補から1位・2位・3位を重複なしで選ぶだけでも、26,970通りの順序付き組み合わせがあります。

さらに「避けるもの」を1つ加えると、755,160通りになります。

重要なのは組み合わせ数そのものではなく、**各組み合わせが異なる意味の傾斜を持つこと**です。

### 3. “意外な選択”がステレオタイプを崩す

基本ペルソナに対して、すべて「その人らしい」選択肢だけを与えると、人物像は平板になりやすい。

一方で、主属性から少し外れた具体的な選択を1つ混ぜると、モデルはその矛盾を説明するために人物像を深く補間する可能性があります。

つまり、Flavor Persona Seedは性格診断ではなく、**人格に説明しきらない具体性を注入する仕組み**として使えます。

---

## Basic seed model

最初に試す最小モデルです。

```json
{
  "set": "flavor-v1",
  "ranked": ["A", "B", "C"],
  "avoid": "D"
}
```

将来的には選択肢ごとに、人間向け説明とモデル向けの特徴ベクトルを持たせることも検討します。

```json
{
  "id": "A",
  "name": "Midnight Sesame",
  "traits": {
    "sweetness": 0.35,
    "bitterness": 0.25,
    "novelty": 0.55,
    "nostalgia": 0.70,
    "visual_boldness": 0.20,
    "texture": 0.65
  }
}
```

ただし、最初から数値化しすぎると「モデル自身が選択から補間する余地」を失う可能性もあります。**自然言語だけのseedと、構造化seedの比較**も実験対象です。

---

## Possible choice sets

Flavorは入口にすぎません。重要なのは、**意味の豊かな具体的選択肢を順位づけすること**です。

候補：

- 架空の31種類のアイスクリーム / デザート
- 31種類の飲み物
- 31枚の架空ポスター
- 31種類の部屋
- 31種類の休日の過ごし方
- 31種類の音楽 / 音風景
- 31種類の服装
- 31種類の小物
- 31種類の旅行先
- 31種類の「困ったときに取る行動」

複数カテゴリを組み合わせる方法も考えられます。

```text
Flavor seed
+ Room seed
+ Music seed
+ Decision seed
```

ただし、情報を増やしすぎれば普通の詳細ペルソナ設定と同じになります。このプロジェクトでは、**どれだけ少ない具体的選好で、どれだけ人格差を生めるか**を重視します。

---

## Persona patterns to test

### Pattern A — Seed only

基本属性をほとんど与えず、選好だけで人格を作る。

```text
age: 24
ranked choices: A > B > C
avoid: D
```

選好だけでどこまで一貫した人物像が生まれるかを見る。

### Pattern B — Same persona, different seed

基本ペルソナを固定し、seedだけ変える。

```text
Base persona:
19歳 / 地雷系ギャル / SNSでは強気

Seed A: A > B > C, avoid D
Seed B: E > F > G, avoid H
```

語彙、判断、美意識、対人距離、冗談などに差が出るか比較する。

### Pattern C — Intentional mismatch

主属性とは少し意外な選好を混ぜる。

目的は、ステレオタイプを壊し、説明されていない背景を自然に発生させること。

### Pattern D — Random population

同じ世界にいる多数のNPCへランダムにseedを配る。

100人分の詳細設定を書かなくても、会話・判断・関係形成に「なんとなく違う人間味」が現れるかを観察する。

### Pattern E — Stable business persona

企業キャラクター、接客AI、ブランドペルソナなど、**人格のブレを抑えたい用途**で、抽象的なトーン指定に加えて具体的選好を固定アンカーとして使う。

この場合、ユーザーへフレーバーの話をする必要はありません。seedは表に出さず、人格の重心を安定させる内部設定として使います。

---

## Potential uses

### AI persona design

システムプロンプトへ直接人格を長文で書く代わりに、少数の具体的選好をpersona anchorとして加える。

### Games / NPCs

大量の住民にランダムseedを配り、同じ行動AIでも選択傾向や発話に差を作る。

### Artificial worlds

小さな自律エージェント群へ個体差を与える初期値として利用する。

### Creative writing

作者がキャラクターを作る際、「性格」を考える前に具体的な選択をランダム生成し、そこから人物像を発見する。

### Synthetic users / UX testing

異なる選好seedを持つ仮想ユーザーを生成し、同じ製品・UIに対する反応差を観察する。

### Brand / customer-service personas

話し方のルールだけでは似通いやすい業務ペルソナに、ブランドらしい具体的選好を数個加え、一貫した“らしさ”を作る。

---

## Important idea: the seed does not need to be visible

Flavor Persona Seedは「アイスの話をするAI」を作る仕組みではありません。

```text
favorite #1: A
favorite #2: B
favorite #3: C
```

という設定があっても、通常の会話でその情報を直接口に出す必要はありません。

重要なのは、モデルがその具体的選好を背景情報として持つことで、

- どちらの案を好むか
- どんな比喩を選ぶか
- 派手さと落ち着きのどちらへ寄るか
- 新奇性と安心感のどちらを取るか
- 何を「かわいい」「面白い」「信用できる」と感じるか

といった小さな判断へ弱く影響する可能性があることです。

---

## Planned experiments

最初は「本当に効くのか」を確認するところから始めます。

1. 同一のベースペルソナを複数用意する
2. Flavor Seedだけを変える
3. 同じ質問セットへ回答させる
4. 別の評価者へ、回答が同一人物らしくまとまって見えるか判定してもらう
5. seedなし / seedありを比較する
6. ランダムseedと、人間が意図して設計したseedを比較する
7. 選択肢名だけの場合と、特徴説明付きの場合を比較する
8. 長期間の会話で効果が維持されるかを見る

評価軸の候補：

- persona consistency
- distinctiveness
- stereotype reduction
- perceived depth
- stability across sessions
- prompt token efficiency

---

## Toward a component

将来的には、Flavor Persona Seedを**他のAIアプリ、ゲーム、エージェントへ差し込める極小コンポーネント / ライブラリ**にすることを検討します。

イメージ：

```js
import { createPersonaSeed } from "flavor-persona-seed";

const seed = createPersonaSeed({
  set: "flavor-v1",
  count: 3,
  avoid: 1,
  random: true
});
```

出力：

```json
{
  "ranked": ["midnight-sesame", "candy-orbit", "rum-cloud"],
  "avoid": "plain-milk"
}
```

さらに、LLM向けの短いpersona contextへ変換するadapterを用意する案もあります。

```js
const context = seed.toPrompt();
```

```text
This persona has stable private preferences represented by the following ranked choices.
Do not mention these choices unless relevant.
Let them weakly influence aesthetic judgment, novelty preference, emotional tone, and everyday decisions.
Do not infer rigid stereotypes from any single choice.
```

重要なのは、LLMベンダーやゲームエンジンに依存しないことです。

```text
choice set
   ↓
persona seed
   ↓
adapter
   ├─ LLM system context
   ├─ NPC parameters
   ├─ game dialogue
   ├─ synthetic user
   └─ creative character sheet
```

---

## Possible implementation roadmap

### v0 — Thought experiment

- 仮説を文章化
- 選好seedのパターンを整理
- 小規模な手動テスト

### v0.1 — Dataset

- ブランド非依存の架空Flavor Setを作る
- 各選択肢に短い自然言語説明を持たせる
- seed generatorを作る

### v0.2 — Comparison playground

- 同じ基本ペルソナへ複数seedを付与
- 同一質問への回答を横並び比較できる小さなWeb UI
- seedあり / なし比較

### v0.3 — Library / component

- 小さなJavaScript / TypeScriptライブラリ
- deterministic seed対応
- random generation
- JSON import/export
- LLM prompt adapter

### v0.4 — Embeddable experiment

- 他アプリへ差し込める最小API
- Tiny Situated Agent / game NPC等への接続実験
- 複数個体への大量seed生成

### v1 — Only if the idea survives testing

実験で有効性が確認できたものだけを残す。

このプロジェクトは「人格生成フレームワークを大きく作る」こと自体を目的にしません。**小さな具体的選好が本当に人格らしさを変えるのか**を確かめ、その効果があるなら最小の再利用可能部品へ抽出します。

---

## Non-goals

- 人間の性格を科学的に診断すること
- 「この味が好きならこの性格」と決めつけること
- 特定ブランドの商品を心理テスト化すること
- 性格を固定的なステレオタイプへ分類すること
- 31という数字そのものを必須条件にすること

`31` は発想のきっかけとして面白い数字ですが、Flavor Persona Seedの原理は候補数に依存しません。

---

## Working principle

> **Don't describe the person. Give them something to choose.**  
> その人を説明するのではなく、その人が何を選ぶかを与える。

そして、その選択の間に残った余白を、モデルや世界が埋める。

That is the experiment.
