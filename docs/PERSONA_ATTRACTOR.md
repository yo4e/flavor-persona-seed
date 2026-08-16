# Persona Attractor Hypothesis

Status: working hypothesis / thought experiment

## Idea

AIの「人格らしさ」は、必ずしも最初から完全な人格設定として列挙されている必要はないのではないか。

長い対話では、明示的な設定に書かれていない好みや判断傾向が、少しずつ一貫して見えることがある。

たとえば、あるAIペルソナが何度も異なる場面で、

- どちらのデザインを好むか
- 何を面白いと感じるか
- どんな人物を信用するか
- どんな色や食べ物へ惹かれるか
- 安全と冒険のどちらへ少し寄るか

といった選択を重ねると、個々の回答は独立していても、後から見ると「この人物ならこちらを選びそう」という領域が見えてくる。

このリポジトリでは、そのような状態を便宜的に **persona attractor（人格の誘引領域）** と呼ぶ。

これは、モデル内部に固定された人格オブジェクトが存在する、あるいはAIが人間と同じ意味で人格・意識を持つ、という主張ではない。

あくまで、**複数の制約・履歴・選択の整合性から、次の出力が特定の方向へ寄りやすくなる現象を説明するための比喩**である。

## What may contribute to an attractor

長期的なAIペルソナでは、少なくとも次のような層が重なって「らしさ」が形成される可能性がある。

```text
base model
    +
system / product constraints
    +
role / name / explicit persona
    +
interaction history
    +
feedback and corrections
    +
previous choices
    ↓
locally consistent response tendency
```

重要なのは、最後の `previous choices`。

「私はこういう性格です」と明示されていなくても、過去に何度も行った具体的な選択そのものが、次の選択を補間する材料になり得る。

## Flavor Persona Seed is the reverse experiment

長期対話で起きることを、極端に圧縮するとどうなるか。

長期対話では、

```text
many conversational choices
    ↓
implicit persona-like attractor
```

が生じるように見える。

Flavor Persona Seedは、その逆方向を試す。

```text
very few concrete preferences
    ↓
initial bias / weak attractor
    ↓
coherent persona-like behavior?
```

つまり、何百回もの会話を待たずに、

```text
Age: 42
Gender: female
Favorite ice cream flavor: Black Sesame
```

のようなごく少数の具体的事実だけで、モデルの人物補間に再現可能な傾斜を与えられるかを試す。

## Why concrete preferences may work

抽象的な形容詞は意味が広い。

```text
calm
intelligent
friendly
```

に対して、具体的な選択は複数の文化的・感覚的連想を同時に持つ。

```text
Favorite ice cream flavor: Black Sesame
```

という一行には、モデルが学習済みの文脈に応じて、味、色、食感、文化、世代、懐かしさ、冒険性、定番性など多くの潜在的な連想が含まれ得る。

そのため、具体的選好は**少ないトークンで多方向の弱い制約を与える圧縮されたpersona anchor**として機能する可能性がある。

ただし、どの連想が生じるかはモデル・言語・文化圏・プロンプトによって変わる。したがって、Flavor Persona Seedは心理学的な性格診断ではなく、LLMの補間特性を利用した生成上の実験である。

## A useful contradiction

人格をすべてステレオタイプ通りに設定すると、人物は平板になりやすい。

一方で、基本属性から少し外れる具体的選好が一つあると、モデルはその組み合わせを成立させるために、説明されていない背景を補間する可能性がある。

例：

```text
Base persona: 19-year-old, flashy fashion, strong social-media persona
Favorite ice cream flavor: a restrained traditional flavor
```

ここで「実は素朴な性格」と説明する必要はない。

説明しないまま具体的選択だけ置くことで、人物の奥行きが出るかを観察する。

## Experiments

同じ質問セットに対して、次を比較したい。

### A. No persona

属性も選好も与えない。

### B. Minimal demographic seed

```text
Age
Gender
```

のみ。

### C. Flavor Persona Seed

```text
Age
Gender
Favorite ice cream flavor
```

のみ。

### D. Explicit adjective persona

```text
calm
adventurous
traditional
...
```

のような通常の性格記述。

### E. Long choice history

明示的な人格設定を置かず、多数の過去の具体的選択を履歴として与える。

### F. Hybrid

短い明示ペルソナ + 少数の具体的選好。

## What to measure

- **Consistency** — 異なる質問でも同一人物らしく見えるか
- **Distinctiveness** — seed違いの人物を区別できるか
- **Recognizability** — 第三者が「この回答はこのキャラっぽい」と当てられるか
- **Stability** — 会話が長くなっても傾向が残るか
- **Stereotype reduction** — 単純な属性ステレオタイプから外れた奥行きが出るか
- **Token efficiency** — 長い人格設定より少ない情報で同程度の一貫性を出せるか
- **Model portability** — 異なるLLMでも同じseedが似た方向の差を作るか

## Business personas

Flavor Persona Seedは、創作キャラクターだけでなく、ブレを減らしたい業務AIにも使える可能性がある。

抽象的なブランドトーンだけでなく、ユーザーには見せない少数の具体的選好を固定すると、モデルが未知の状況に遭遇したときにも「この人格ならどちらを選ぶか」を補間する足場になるかもしれない。

ここで重要なのは、フレーバーそのものを会話に出すことではない。

**選好は背景のanchorであり、回答内容ではない。**

## Connection to Tiny Situated Agents

人工世界やゲームの住民では、人格を静的な設定として与えるだけでなく、住民自身が世界の中で選択を積み重ね、その履歴から個体差が強くなっていく設計も考えられる。

```text
initial Flavor Persona Seed
        ↓
small initial preference bias
        ↓
choices made in the world
        ↓
preference history / memory
        ↓
stronger individual tendency
```

この場合、Flavor Persona Seedは「完成した人格」ではなく、**人格が育ち始めるためのごく小さな初期条件**になる。

---

**Working hypothesis:** personality-like consistency may be produced not only by describing a character, but also by accumulating — or seeding — meaningful choices.
