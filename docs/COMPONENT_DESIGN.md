# Flavor Persona Seed — Component Design Notes

Status: idea / pre-implementation

## Goal

Flavor Persona Seedを、LLM、ゲーム、人工世界、創作支援ツールなどへ差し込める**極小・環境非依存のpersona seed component**として実装する場合の設計メモ。

このコンポーネント自身は「人格を完成させるAI」にはしない。

役割は小さく保つ。

```text
meaningful choices
      ↓
rank / avoid / weight
      ↓
persona seed
      ↓
application-specific adapter
```

## Core responsibilities

Coreが担当するもの：

- choice setの読み込み
- 順位付き選好の生成
- avoid / dislikeの生成
- deterministic random seed
- JSON serialize / deserialize
- seed IDの生成
- 簡単なvalidation

Coreが担当しないもの：

- LLM API呼び出し
- キャラクター本文の自動生成
- 会話履歴管理
- 長期記憶
- UI
- ゲームAI
- 心理診断

## Proposed data model

### Choice set

```json
{
  "id": "flavor-v1",
  "version": 1,
  "choices": [
    {
      "id": "midnight-sesame",
      "label": "Midnight Sesame",
      "description": "Dark roasted sesame, low sweetness, dense texture."
    }
  ]
}
```

特徴量は任意。

```json
{
  "traits": {
    "sweetness": 0.35,
    "novelty": 0.55,
    "nostalgia": 0.70
  }
}
```

最初の実験では、traitsなしの自然言語版も必ず残す。

### Persona seed

```json
{
  "schema": "fps-1",
  "choiceSet": "flavor-v1",
  "ranked": [
    "midnight-sesame",
    "candy-orbit",
    "rum-cloud"
  ],
  "avoid": ["plain-milk"],
  "randomSeed": "a1f39c"
}
```

## Proposed API

```js
import {
  loadChoiceSet,
  createPersonaSeed
} from "flavor-persona-seed";

const set = loadChoiceSet(flavorSet);

const personaSeed = createPersonaSeed(set, {
  ranked: 3,
  avoid: 1,
  seed: "resident-0042"
});
```

同じchoice setと同じ`seed`なら、常に同じpersona seedを返せるようにする。

これはNPCや人工世界で重要。

```text
resident-0042
      ↓
always same ranked preferences
```

## Adapters

Coreと利用先を分離する。

### Prompt adapter

```js
import { toPromptContext } from "flavor-persona-seed/adapters/prompt";

const context = toPromptContext(personaSeed, set);
```

出力例：

```text
The character has stable private preferences.
Ranked preferences:
1. Midnight Sesame — dark, roasted, restrained sweetness.
2. Candy Orbit — playful, bright, surprising texture.
3. Rum Cloud — aromatic, mature, slightly nostalgic.
Avoids: Plain Milk.

Treat these as weak personality anchors, not deterministic personality rules.
Do not mention them unless relevant.
```

### NPC adapter

```js
const npcBias = toNpcBias(personaSeed, set);
```

ゲーム側では、会話用文章ではなく数値パラメータへ落とすこともできる。

```json
{
  "novelty": 0.62,
  "nostalgia": 0.71,
  "visualBoldness": 0.44,
  "comfortSeeking": 0.53
}
```

ただし、これを唯一の正解とはしない。自然言語から生まれる曖昧さ自体がPersona Seedの重要部分かもしれないため。

### Character-sheet adapter

創作向け。

seedから「性格」を断定するのではなく、作者が考えるための問いを返す。

```text
This character strongly prefers A over B and C, but avoids D.
What past experience might make that combination feel natural?
Which preference do they hide from others?
```

## Embedding forms

実装後は用途に応じて複数の配布形態を検討する。

### JavaScript / TypeScript library

最優先候補。

```bash
npm install flavor-persona-seed
```

小さく、依存を極力持たない。

### Web component

seedを生成・確認する小さなUIとしてはWeb Component化も可能。

```html
<flavor-persona-seed
  set="flavor-v1"
  ranked="3"
  avoid="1">
</flavor-persona-seed>
```

ただし、人格seedの主用途はUIではなくデータなので、Web ComponentはCoreとは分離する。

### CLI

大量NPC生成や実験用。

```bash
fps generate --set flavor-v1 --count 100 --seed mintwhirl
```

### Static browser demo

APIキー不要でseed生成・比較だけできるGitHub Pagesデモ。

LLM評価を行う場合も、最初から特定APIへ固定しない。

## Population generation

人工世界への利用では、個体IDから決定論的にseedを作れると便利。

```js
const residents = Array.from({ length: 100 }, (_, i) => {
  return createPersonaSeed(set, {
    ranked: 3,
    avoid: 1,
    seed: `resident-${i}`
  });
});
```

これにより、セーブデータへ人格設定全文を持たなくても、同じworld seedから同じ住民の初期選好を再現できる可能性がある。

## Multiple choice-set layers

将来的には複数カテゴリを重ねることもできる。

```json
{
  "flavor": {
    "ranked": ["A", "B", "C"],
    "avoid": ["D"]
  },
  "room": {
    "ranked": ["R7", "R2", "R19"]
  },
  "music": {
    "ranked": ["M11", "M4", "M8"]
  }
}
```

ただし、最小性を失わないようにする。

目標は「詳細設定を増やすこと」ではない。

**少ない選択から人格差が生まれる最小点を探すこと。**

## Random vs authored seed

2つの用途を同等に扱う。

### Random

大量NPC、synthetic users、実験群。

### Authored

ブランドペルソナ、重要キャラクター、対話エージェント。

```js
createPersonaSeed(set, {
  ranked: ["A", "F", "C"],
  avoid: ["Q"]
});
```

## Design principles

### Weak influence

seedは人格を完全決定しない。

### Concrete over adjective

可能な限り「大胆」ではなく「何を選ぶか」を置く。

### Keep contradictions

矛盾はエラーではなく、人格の奥行きになり得る。

### Portable

特定LLM、API、ゲームエンジンに依存しない。

### Tiny

巨大なpersona frameworkへ育てることを目的にしない。

### Reproducible

研究・ゲーム用途では同じseedから同じ初期選好を再現できる。

## First implementation milestone

最初に必要なのはこれだけ。

```text
choice-set.json
seed-generator.ts
serializer.ts
one prompt adapter
small browser demo
```

それで効果がなければ、そこで終わってよい。

効果があるなら、そのとき初めて別カテゴリ、NPC adapter、Web Component等へ拡張する。
