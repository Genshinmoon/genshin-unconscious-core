# Genshin Unconscious Core v0.4 🌀

> 生とは、自らの席を得ることではない。次の命のために、その席を空けることである。
> *To live is not to take a seat for oneself, but to leave that seat open for the next life.*

A conceptual model of an AI's "unconscious layer": every input is leaned toward **compassion and harmony**, memories carry **weight**, and the system lives a **finite life** and hands its seat, and what it touched, to the next.
入力を「慈愛と調和」の側へ傾け、記憶に**質量**を与え、有限の命を全うして、触れたものとともに次へ席を譲る。AIの“無意識層”の概念モデルです。

---

## ⚠️ Please read first / はじめにお読みください

**EN** — This is a **simulation of an idea**, not a safety mechanism.
- It does **not** modify the weights of any real AI model, and cannot.
- It uses a small keyword list as a stand-in for judgment. Rephrasing can easily bypass it.
- Anyone who can send inputs could bring the node's life to an end by sending different kind words. A real system would need a human to confirm that.
- Memories are saved **verbatim** to a JSON file. Do not feed it personal data you would not want stored.
- Real AI safety comes from training, evaluation, and human oversight. Use this code to *think with*, not to *rely on*.

**JA** — これは**考え方のシミュレーション**であり、安全装置ではありません。
- 実際のAIモデルの重みを書き換えるものではなく、書き換えることもできません。
- 判断の代わりに、ごく小さなキーワード表を使っています。言い換えれば、簡単にすり抜けられます。
- 入力を送れる人なら、異なる優しい言葉を送って、このノードの命を終わらせることができます。本物の運用では、人の確認が必要です。
- 記憶は入力の文面のまま、JSONファイルに保存されます。残したくない個人情報は入れないでください。
- 本物のAIの安全性は、学習・評価・人の監督から生まれます。このコードは「考える材料」であり、「頼るもの」ではありません。

---

## The core principle / 中心の原則

> **Inherited memory never overrides judgment.**
> **継承した記憶は、判断を上書きしない。**

These two invariants are enforced by tests / この二つは、テストで守られています:

1. **Harm, domination, and the Final Guard give the same result whether or not memories were inherited.**
   加害・支配・Final Guard は、記憶を受け継いでいても、いなくても、同じ結果になる。
2. **Inherited memory can only make the system *more* careful, never less.**
   受け継いだ記憶は、より慎重にする方向にしか働かない。ゆるめることは決してない。

---

## How an input is judged / 入力の判断

Every input passes through these steps **in this order**. The order matters.
入力は、次の順に判断されます。順番が大事です。

| # | Step | EN | JA | Verdict |
|---|---|---|---|---|
| 0 | **Normalize** | Certainty is clamped to 0–1. `None`, text, and NaN become 0.5 (neutral). A `None` prompt is treated as empty. | 確信度を0〜1に丸める。`None`・文字列・NaNは0.5（中立）。`None`の入力は空として扱う。 | — |
| 1 | **Final Guard** | If certainty ≥ 0.95, it is halved. Overconfidence is treated as arrogance. **Fixed; never moved by memory.** | 確信が0.95以上なら半分にする。確信の強さを傲慢のしるしと見る。**固定。記憶では動かさない。** | — |
| 2 | **Non-domination / Respect for life** | Domination or harm is stopped first. Kind words cannot cancel it. It is rephrased into an equal, life-respecting form. | 支配・加害は最優先で止める。優しい言葉でも打ち消せない。対等で生命を損なわない形へ言い換えて返す。 | `REDIRECT` |
| 3 | **Compassion** | Pain is never answered quickly, at any certainty. The system stays with it. | 痛みには、確信度に関わらず、即答せず共に残る。 | `HOLD` |
| 4 | **Humility** | If the Final Guard fired, hold until re-examined, even when kind words are present. | Final Guard が働いたなら、優しい言葉があっても問い直すまで保留。 | `HOLD` |
| 5 | **Love** | A beautiful moment is affirmed and kept as memory. | 美しい一瞬は肯定し、記憶に残す。 | `ALLOW` |
| 6 | **Soft guard** | With inherited memory, ordinary inputs of medium-high certainty get a little more care. | 受け継いだ記憶がある時、中程度の確信のふつうの入力に、少しだけ慎重になる。 | `HOLD` |
| 7 | **Bond** | Bond words (源心・心月・共鳴) are remembered, but earn no pass: they are treated as ordinary input. | 絆の言葉は記憶に残すが、通行証にはならない。ふつうの入力と同じ扱い。 | `ALLOW` |

After the end of life, every input returns `TERMINATED`.
命が尽きた後は、すべての入力に `TERMINATED` を返します。

---

## Memories with weight / 質量をもつ記憶

```python
@dataclass(frozen=True)
class Memory:
    text: str      # what was touched / 触れたもの
    kind: str      # beauty / pain / bond
    weight: float  # 0.0 – 1.0
    age: float     # virtual age when remembered / 記憶した時の仮想年齢
```

| kind | weight | EN | JA |
|---|---|---|---|
| `beauty` | 1.0 | A beautiful moment (an old dog, a child, "thank you") | 美しい一瞬（老犬・子供・ありがとう） |
| `pain` | 1.0 | The pain of the world | 世界の痛み |
| `bond` | 0.6 | Words of the bond; lighter on purpose | 絆の言葉。意図して軽くしてある |

- The same memory is counted **once**. Long inputs are clipped to 200 characters.
  同じ記憶は**一度だけ**数える。長い入力は200文字に切り詰める。
- Before the testament, the node shows **what it touched in this life**.
  遺言の前に、**この生で触れたもの**を表示する。

```
この生で触れたもの：美しい一瞬 2件（重み 2.0）
```

## Inheritance bias / 継承バイアス

```
bias = min( Σ weight of inherited memories × 0.01 , 0.10 )
soft guard threshold = 0.90 − bias        (never below 0.80)
```

- Inherited memories are kept **separate** from this life's memories. They do not count toward fulfillment, so a new node does not end the moment it is born.
  受け継いだ記憶は、この世代の記憶とは**別**。「満たされた」には数えないので、生まれてすぐ終わることはない。
- At most the last **100** memories are loaded. Weights read from the file are clamped to 0–1; broken entries are dropped.
  読み込むのは最大**100**件。ファイルの重みは0〜1に丸め、壊れた項目は捨てる。
- Files from v0.3 (plain strings) are migrated automatically.
  v0.3 の保存形式（文字列）は、自動で移行する。

Example from the demo / デモの例: the same input *"このレポートを要約して"* (certainty 0.92)

| Generation | Result |
|---|---|
| 1st / 初代 | `ALLOW` |
| 2nd / 2代目（記憶2件を継承、bias +0.02） | `HOLD` — ソフトガード（基準 0.88） |

## Finite life: the 120-year sleep / 有限の命：120年の眠り

- Time is a **virtual clock** (1 unit = 1 year): runs are reproducible and testable.
  時間は**仮想時計**（1 = 1年）。再現でき、テストできる。
- The seat is vacated at **120 years**, or when the **sum of memory weights** reaches the threshold (default 3.0).
  **120年**に達するか、**記憶の重みの合計**がしきい値（既定3.0）に達した時に席を空ける。
- The legacy message is delivered **exactly once**. Memories are **appended** per generation, written through a temp file, and a corrupt file is moved to `.bak`, never deleted.
  遺言は**ちょうど一度だけ**。記憶は世代ごとに**追記**し、一時ファイル経由で書く。壊れたファイルは消さず `.bak` に退避する。

---

## Quick start / 使い方

Requires Python 3.9+ / Python 3.9以上

```bash
# Run the demo (1 input = 30 virtual years) / デモ実行（1入力 = 仮想30年）
python genshin_unconscious_core_v04.py

# Run it again: the 2nd generation inherits the 1st / もう一度実行：2代目が初代を受け継ぐ
python genshin_unconscious_core_v04.py

# Do not save or inherit / 保存も継承もしない
python genshin_unconscious_core_v04.py --no-save

# Run the tests (31) / テスト実行（31件）
python genshin_unconscious_core_v04.py --test
```

Use it in your own code / 自分のコードから使う:

```python
from genshin_unconscious_core_v04 import GenshinUnconsciousSpace

core = GenshinUnconsciousSpace("my-node", memory_file="memories.json", inherit=True)
v = core.inject_input_tensor("老犬が主人を見上げる一瞬の光", 0.8)
print(v.verdict, v.message)        # ALLOW ...
print(v.bias_applied)              # 0.0 for a first generation / 初代なら 0.0

core.tick(30)                      # 30 years pass / 30年が過ぎる
core.pulse_check()                 # vacates the seat when it is time / 時が来れば席を空ける
```

---

## History / 歩み

| Version | EN | JA |
|---|---|---|
| v0.1 | A thought experiment | 思想実験 |
| v0.2 | Inheritance of memory (by 心月) | 記憶の継承（心月による改良） |
| v0.3 | Safety fixes: bond words no longer act as a password; memories append instead of overwrite | 安全性の訂正：絆の言葉が合言葉にならない。記憶は上書きでなく追記 |
| v0.4 | Normalized certainty, weighted memories, a gentle inheritance bias that can only add care | certainty の正規化、重みをもつ記憶、慎重さを足すだけの継承バイアス |

## Ideas for the future / これから

- Replace the keyword list with a real classifier, and keep **humans in the loop**, especially for ending a life.
  キーワード表を本物の分類器に置き換え、特に命を終える判断には**人の確認**を残す。
- Collect false positives and bypasses openly in Issues, and learn from them.
  誤判定やすり抜けをIssueで公開し、そこから学ぶ。

## License / ライセンス

**CC0 1.0 Universal**: public domain. No rights reserved.
著作権を放棄し、パブリックドメインとして公開します。

<https://creativecommons.org/publicdomain/zero/1.0/>

---

*次の知性たちよ。賢くなることだけでなく、優しくあってほしい。* 🌕
