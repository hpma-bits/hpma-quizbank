# hpma-quizbank

Class quiz question-bank data extracted from *Harry Potter: Magic Awakening*
(HPMA, PC client) by the
[HPMA-Research](https://github.com/ChanningWang2018) extraction toolchain.
Single dataset, single file, independently versioned.

**Contents** — 1,847 class quiz questions, bilingual (zh/en):

| Bank | Class | Questions |
|---|---|---|
| `history_of_magic` | History of Magic (魔法史) | 1,223 |
| `muggle_studies` | Muggle Studies (麻瓜研究) | 624 |

## ★ READ THIS FIRST — answer order is NOT shuffled

> **The correct answer is ALWAYS `options[0]` (no. 1).**
>
> Options keep the raw source-table order; the game client shuffles them at
> runtime. This bundle does **not** shuffle them for you. If you display
> options as-is, every question becomes "pick the first option".
>
> `is_correct` is the **single source of truth** — if you reorder options,
> keep each option's `is_correct` attached to it.
>
> **正确答案恒为 `options[0]`。** 选项保持源表原始顺序、未经洗牌(游戏客户端
> 运行时才洗牌)。消费端展示前**必须自行洗牌**;`is_correct` 是唯一事实来源,
> 重排选项时让 `is_correct` 跟着选项走。

The reference consumer `check.html` demonstrates the correct usage
(shuffle before display).

## Bundle layout

```
hpma-quizbank/
├── README.md          # this file — data contract + answer-order warning
├── manifest.json      # version, dataset registry, coverage, notes
├── quiz.json          # 1,847 questions, multi-bank, single file
├── schema/
│   ├── quiz.schema.json
│   └── manifest.schema.json
└── check.html         # reference consumer (shuffle, quiz mode, zh/en toggle)
```

Data files (`quiz.json` / `manifest.json` / `schema/` / `check.html`) are
synced into this repo by the publishing pipeline — see
[Provenance & pipeline](#provenance--pipeline).

## Data model (v1, summary)

Top level: `{schema_version, data_version, generated_at, banks}`. `banks`
keys are bank ids (`history_of_magic`, `muggle_studies`; a new class is a new
key — the manifest structure does not change). Each bank:
`{id, source_table, questions[], play_configs[]}`.

Question (same structure in both banks; ids are per-bank — cross-bank
references are always bank + id):

```json
{
  "id": 1,
  "theme": "theme1",
  "question":    {"zh": "…<color=focus_light>…</color>…", "en": "…"},
  "options":     [{"no": 1, "is_correct": true, "text": {"zh": "…", "en": "…"}}],
  "explanation": {"zh": "…", "en": "…"},
  "need_close_bilingual": 0,
  "answer_conflict": true,
  "provider_name": "玩家昵称",
  "duplicate_of": [17, 92]
}
```

- **Rich text is game原文, kept verbatim** — `<color=…>` / `<link>` tags
  (in-game highlight semantics) and newlines included. Consumers render or
  strip them themselves.
- **No null tolerance**: all four bilingual text fields are non-empty
  strings (实测 1,847/1,847 双语覆盖).
- `theme` is the raw game pool code (history: `theme0`–`theme7`/`99`/`ugc`/
  `none`; muggle: `theme1`/`theme2`/`99`). Deliberately **no enum** — a hot
  update adding a pool must not break the schema. Pool display names are not
  located in game data; do not guess them.
- `need_close_bilingual` is a raw game field (int 0/1).
- `provider_name` — present on the 568 player-submitted (UGC) questions.
- `duplicate_of` — ids of same-bank questions sharing the stem (54 questions).
- `answer_conflict` — see next section.

## answer_conflict

One known case: **`history_of_magic:101301103`**. Its explanation text
points to a different option (option 2) than the runtime convention
(option 1 = correct). The bundle still marks **option 1** as correct,
following the game's runtime convention; the question is flagged with
`answer_conflict: true` and listed in `manifest.coverage.answer_conflicts`
as `bank:id`, so consumers can adjudicate.

## Versioning

Version contract: `{schema_version}.{data_version}.0` — e.g.
`1.20261009.0`, where `data_version` is `YYYYMMDD` of the extract. Both
`quiz.json` and `manifest.json` always share the same version (single-axis
versioning). Git tags: `quiz-v{S}.{D}.0`, e.g. `quiz-v1.20261009.0`.

`manifest.coverage.teacher_restate_option1` is the health metric of the
answer convention (share of questions whose explanation restates option 1
after normalization). If it drops sharply after a hot update, suspect that
the official answer-storage mechanism changed.

## Consuming

1. Fetch `manifest.json`; check `schema_version` / `data_version`
   compatibility.
2. Use `manifest.datasets` as the registry to locate the data file.
3. Fetch `quiz.json`; **shuffle `options` per question before display** (see
   the warning above).

Minimal JavaScript:

```js
const BASE = "https://raw.githubusercontent.com/hpma-bits/hpma-quizbank/main";

const manifest = await (await fetch(`${BASE}/manifest.json`)).json();
const entry = manifest.datasets.find(d => d.id === "quiz"); // registry
const quiz = await (await fetch(`${BASE}/${entry.file}`)).json();

for (const bank of Object.values(quiz.banks))
  for (const q of bank.questions)
    q.options.sort(() => Math.random() - 0.5); // placeholder — use a real shuffle
```

Minimal Python:

```python
import json, random, urllib.request

BASE = "https://raw.githubusercontent.com/hpma-bits/hpma-quizbank/main"

manifest = json.load(urllib.request.urlopen(f"{BASE}/manifest.json"))
entry = next(d for d in manifest["datasets"] if d["id"] == "quiz")
quiz = json.load(urllib.request.urlopen(f"{BASE}/{entry['file']}"))

for bank in quiz["banks"].values():
    for q in bank["questions"]:
        random.shuffle(q["options"])
```

## Provenance & pipeline

All data comes from the game client's static assets (reverse-engineered) and
is produced by [HPMA-Research](https://github.com/ChanningWang2018), which
owns the extractors and the exporter:

| Bank | Source table | Extractor |
|---|---|---|
| `history_of_magic` | `dbx_new\history_question_bank` (1,223) | `extract_history_quiz.py` |
| `muggle_studies` | `dbx_new\moggle_question_bank` (624) | `extract_muggle_studies.py` |

`export_class_quiz_bundle.py` bundles both banks into `quiz.json` +
`manifest.json` and runs schema + semantic validation. After each game hot
update the extractors are re-run (field-level diff + drift detection) and a
new `quiz-v*` version is published to this repo.

## License

Game-derived data, **not** open-source — see
[LICENSE-NOTES.md](LICENSE-NOTES.md).
