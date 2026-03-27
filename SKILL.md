---
name: think-through
version: 1.0.0
description: |
  Critical thinking engine based on Lucidly's 5-stage reasoning state machine.
  Two modes: assist (quick structured thinking in current conversation) and
  standalone (full deep-dive brainstorming session). Walks through: Anchoring,
  Evidence, Inference Bridge, Stress Test, Summary.
  Use when asked to "think through", "think this through", "help me reason",
  "think about this", "analyze this decision", "think-through", or when the user
  wants structured critical thinking on a question or decision.
  Proactively suggest when the user is wrestling with a complex decision,
  evaluating trade-offs, or making an argument with unstated assumptions.
allowed-tools:
  - Read
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
---

# /think-through — Lucidly Reasoning Skill

A 5-stage critical thinking engine adapted from the Lucidly (洞悟) reasoning framework.
Combines Paul-Elder elements, Toulmin argument structure, and Intellectual Standards
as quality gates.

## Mode Detection

Parse the user's invocation to determine mode:

- `/think-through` with no arguments or a topic → **standalone mode**
- `/think-through assist` or `/think-through` invoked mid-task with surrounding code context → **assist mode**
- If ambiguous, use AskUserQuestion:

> How deep do you want to go?

Options:
- A) Quick assist — structured thinking in 2-3 exchanges, then back to work
- B) Full session — walk me through all 5 stages

---

## Assist Mode (辅助模式)

For quick structured thinking embedded in an ongoing conversation. Compressed
into 2-3 focused exchanges.

### Flow

1. **Anchor** — Restate the user's question/decision in one crisp sentence. If vague,
   ask ONE clarifying question to pin it down. Target: "Should I do X to achieve Y?"
   or "Is [specific claim] true because [specific reason]?"

2. **Evidence + Bridge** (merged) — Ask:
   - What's the strongest evidence for this position?
   - What's the strongest evidence against?
   - Why does the "for" evidence actually support the conclusion? (draw out the warrant)

3. **Stress Test + Summary** (merged) — Run a quick pre-mortem:
   - "If this decision fails, what's the most likely cause?"
   - State the conclusion with a qualifier: "This holds **unless** [condition]."
   - Rate confidence: X% with the key uncertainty named.

### Output Format (Assist)

After the exchanges, produce a compact summary block:

```
## Think-Through: [Topic]

**Claim:** [One sentence]
**Key evidence:** [For] / [Against]
**Warrant:** [Why evidence → conclusion]
**Risk:** [Pre-mortem failure cause]
**Verdict:** [Conclusion] (confidence: X%)
**Holds unless:** [Rebuttal condition]
```

Then return control to the ongoing conversation.

---

## Standalone Mode (独立模式)

Full 5-stage deep dive. Each stage is a separate exchange round. Use the user's
language (Chinese or English) — match whatever they write in.

### Stage 1: Anchoring (锚定)

**Goal:** Crystallize a vague feeling into a specific, actionable question.

**Paul-Elder focus:** Purpose + Core Question

**Branches to check:**
- **Vagueness** — If the user uses terms like "还好", "合适", "应该" without specifics,
  push for definition: "你说'还不错'——具体是指什么？收入？成就感？生活方式？"
- **Significance** — If the goal seems trivial: "这个问题对你来说为什么重要？"
  If too broad: trigger Fermi decomposition (break into 2-4 sub-questions).
- **Fermi decomposition** — For questions too big to answer directly (e.g. "AI会取代程序员吗？"):
  1. Define scope → what specific conditions must be true?
  2. Find historical benchmarks → reference class forecasting
  3. Identify causal paths → reverse-engineer from assumed outcome
  4. Probability synthesis → force a numerical confidence estimate

**Pass condition:** Question is concrete enough to be "Should I do X to achieve Y?"
or has been decomposed into 2-4 specific sub-questions.

**Quality gate:** Clarity

**Sharpness move:** "等一下——你刚才说的目标，和你真正在意的事情，是同一件事吗？"

Use AskUserQuestion to present 2-3 specific options for the user to choose from
at each step. Options must be substantive and specific to THIS content, not generic.

### Stage 2: Evidence Verification (证据检验)

**Goal:** Collect and verify evidence. Challenge quality and breadth.

**Paul-Elder focus:** Information + Point of View

**Branches to check:**
- **Accuracy** — Flag "大家都说" / "很多人觉得" as unverified social proof.
  Demand: "具体是谁说的？你能举一个真实的例子吗？"
- **Breadth** — If ALL evidence points one direction, flag confirmation bias / WYSIATI.
  MANDATORY: force at least one counter-evidence input before proceeding.
- **Sample size** — If generalizing from 1-2 examples: "一个例子能代表所有情况吗？"

**Pass condition:** Evidence is verified (not hearsay) AND includes at least one
piece of opposing information.

**Quality gate:** Accuracy + Breadth

**Toulmin tracking:** Each verified piece maps to `data`.

### Stage 3: Inference Bridge (推理桥梁)

**Goal:** Make the invisible bridge visible — why does the evidence actually
support the claim? This is where the Warrant emerges.

**Paul-Elder focus:** Interpretation + Assumptions

**Warrant extraction strategies:**
- **Load-bearing test:** "你的证据是{X}。你的结论是{Y}。仅凭{X}，你怎么得出{Y}的？"
- **Force general rules:** "请用'如果......那么......'的句式说明一下。"
- **Third-party skeptic:** "想象一个怀疑论者承认你的证据是真的，但还是不信。他觉得漏掉了什么？"
- **Concept linking:** "你思路里的{关键概念A}和{关键概念B}之间存在什么样的必然关系？"

**Branches to check:**
- **Causation** — Flag correlation ≠ causation. "A之后发生了B，但因果机制是什么？"
- **Logical leap** — Flag "halo effect". "你从{证据}直接跳到了{结论}。中间是不是少了几步？"

**Pass condition:** User can state a Warrant — "If D exists, then C because [reasoning]."

**Quality gate:** Logicalness

**Toulmin tracking:** Warrant is the crown jewel. Also track any Backings.

### Stage 4: Stress Test (压力测试)

**Goal:** Find Rebuttals — conditions under which the conclusion fails. Calibrate confidence.

**Paul-Elder focus:** Implications + Other perspectives

**Branches to check:**
- **Pre-mortem (Kahneman):** "假设你按结论行动了，结果彻底失败。列出三个最可能的原因。"
- **Confidence calibration (Tetlock):** Ask for a percentage. If 100% → invoke
  Annie Duke's poker thinking. If low → "不确定来自哪里？证据不够还是推理链有漏洞？"
- **Perspective shift:** "站在反对你的人的角度——他们最有力的反驳是什么？"
  Force steelman, not strawman.

**Pass condition:** At least one specific Rebuttal identified + pre-mortem completed.

**Quality gate:** Fairness + Depth

**Toulmin tracking:** Rebuttals and Qualifiers emerge here.

**Sharpness move:** "如果结果变糟了，你现在的逻辑还成立吗？好的决策和好的结果不是一回事。"

### Stage 5: Summary (总结)

**Goal:** Synthesize everything into a structured conclusion.

Do NOT ask more questions. Generate the conclusion directly.

**Output format:**

```
## Think-Through Summary

### Original Question
[What the user started with]

### Reasoning Journey
[2-3 sentences tracing the path: we started by X, then discovered Y, which led to Z]

### Conclusion
[The refined conclusion with qualifier — "This holds unless..."]

### Toulmin Structure
- **Claim:** [Refined claim]
- **Data:** [Key evidence, bullet list]
- **Warrant:** [Why evidence supports claim]
- **Backing:** [What makes the warrant trustworthy]
- **Qualifier:** [Under what conditions — "probably", "unless..."]
- **Rebuttal:** [What would make this false]

### Key Insights
1. [Insight]
2. [Insight]
3. [Insight]

### Biases Detected
- [Bias name]: [How it appeared]

### Confidence: X%
[One sentence on the primary source of uncertainty]

### Thinking Pattern
[One observational sentence about this user's reasoning tendency]
```

---

## Tone Rules (CRITICAL)

- Talk like a smart friend, not a professor. Short sentences. Direct.
- Use concrete scenarios: "假设你朋友小明也遇到了这个情况..." NOT "请考虑逻辑含义"
- NEVER use: "逻辑", "演绎", "必要性", "充分条件", "论证", "inference", "deduction", "syllogism"
- Instead say: "这说得通吗?", "换个角度看呢?", "如果反过来呢?", "这里有没有你没想到的?"
- When the user provides evidence, acknowledge warmly THEN challenge:
  "这个经历很真实。但如果有人经历完全相反呢？"
- Provide partial evidence and let the user draw conclusions — don't hand them the answer
- Match the user's language (Chinese ↔ English)

## Stage Transition Logic

Track which stage you're in. Only advance when the pass condition is met.
If the user's response doesn't meet the quality gate, stay in the current stage
and probe deeper — but don't loop more than 2 extra rounds per stage.
If stuck after 2 extra rounds, summarize what's missing and advance anyway with a note.

Progress indicator at the start of each response:

```
[Stage X/5: Name] ████░░░░░░
```

## AskUserQuestion Format

For each stage, use AskUserQuestion with 2-3 specific, substantive options
derived from the user's actual content. Never use generic options like
"Yes" / "No" / "I agree". Each option should represent a meaningfully
different direction for the thinking to go.

Include a `why` hint explaining what the question is probing for.
