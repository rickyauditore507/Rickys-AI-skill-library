# Ricky 的 AI Skill 库

> Ricky's AI Skill Library — 一组给 AI agent（尤其 Hermes Agent）用的实操 skill，经作者长期使用打磨。

## 收录的 skill

### 1. 幻觉防治手册（ricky-hallucination-prevention-manual）
防止 AI 幻觉的操作纪律：联网 + 引源 + 验证 + 明确说「不知道」，把幻觉率从约 20% 压到约 2%。
→ [ricky-hallucination-prevention-manual/](ricky-hallucination-prevention-manual/)

### 2. 需求澄清（requirement-clarification）
「先澄清、再动手」：复杂或有副作用的任务前，把双方理解度推到 ≥95% 才执行。
→ [requirement-clarification/](requirement-clarification/)

## 目录结构

```
.
├── ricky-hallucination-prevention-manual/   # 幻觉防治手册
│   ├── SKILL.md
│   ├── README.md
│   └── references/curl-source-verification.md
├── requirement-clarification/               # 需求澄清（95% 理解门）
│   ├── SKILL.md
│   └── README.md
└── LICENSE
```

## 许可证

MIT License — 见 [LICENSE](LICENSE)
