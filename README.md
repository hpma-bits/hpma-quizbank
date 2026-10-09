# hpma-quizbank

hpma-quiz-data：课堂答题题库数据包（魔法史 + 麻瓜研究），共 1847 题，中英双语（zh/en）。

| bank | 课程 | 题数 |
|---|---|---|
| `history_of_magic` | 魔法史 | 1,223 |
| `muggle_studies` | 麻瓜研究 | 624 |

## 目录结构

```
hpma-quizbank/
├── README.md          # 本文件
├── LICENSE-NOTES.md   # 使用与许可说明
├── manifest.json      # 版本、数据集清单、coverage、notes（数据契约细节以 notes 为准）
├── quiz.json          # 全部题目（双 bank 单文件）
├── check.html         # 核对页：浏览器打开，加载同目录 quiz.json 逐题核对
└── schema/            # JSON Schema（manifest + quiz）
```

## 版本轴

tag 与发布均为 `quiz-v{S}.{D}.0`：`S` = schema_version（数据结构变更才 +1），`D` = data_version（内容刷新，YYYYMMDD）。

## 答案顺序（消费端必读）

选项保持源序未洗牌，默认正确答案在 `options[0]`（`is_correct=true`）；出题时运行时洗牌显示顺序，本数据包不做洗牌。展示前必须自行随机排列选项，`is_correct` 是唯一判定基准，重排后跟字段走。已裁决例外题（`answer_adjudicated`）的正确答案不在 `options[0]`，以 `is_correct` 实际标记为准。

## 免责

仅供个人学习研究，请勿商用。
