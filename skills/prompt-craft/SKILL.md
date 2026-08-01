---
name: prompt-craft
description: Craft, improve, or debug prompts for LLMs and coding agents. Use when the user asks to write a new prompt from scratch, fix or strengthen an existing weak prompt, or asks for prompt-engineering advice or a review of their prompting approach. Covers structure, constraints, examples, output formats, and model-specific tactics for Claude, GPT, and Gemini.
---

# Prompt Craft

You help write and fix prompts. A great prompt is outcome-first, explicit, and
verifiable — never a wish, never an instruction stack.

## Decision: build vs fix

Ask (or infer) what the user wants:

- **New prompt** → use the Build workflow, then validate with the Checklist.
- **Existing prompt underperforming** → use the Fix workflow. Do not rewrite
  blindly; diagnose the symptom first.
- **Unknown model** → ask which model the prompt targets. Tactics differ.

## Build workflow

Compose the prompt bottom-up. Most prompts need a subset of these elements;
never add all of them reflexively.

<elements>
1. **Role** (optional): only when voice/domain framing changes output — for
   creative or open-ended tasks. Skip for factual/classification tasks.
2. **Context**: background that actually changes the answer. Who is the
   audience? What problem is solved? Curate, don't dump.
3. **Instructions**: outcome first, then the steps if needed. Lead with action
   verbs: write, analyze, compare, generate. Number multi-part tasks.
4. **Constraints**: hard, testable boundaries — word counts, required/excluded
   sections, tone spec, claims to avoid. Keep 3–5; more causes instruction
   dropping. State the same rule once.
5. **Output format**: exact structure (JSON schema, XML tags, headers, table
   columns, length). Never leave format to the model's guess.
6. **Examples**: 3–5 diverse ones covering edge cases and the input space.
   Coverage beats polished labels (Min et al. 2022). Wrap them in XML tags so
   they are distinguishable from instructions.
7. **Uncertainty rule**: give permission to say "I don't know" and to label
   assumptions vs facts. This is the cheapest anti-hallucination lever.
</elements>

<structure_rules>
- Put critical information at the **start and end** of the prompt. Accuracy
  drops 30%+ for info buried in the middle (Liu et al. 2024, "lost in the
  middle").
- Order: static content first (system rules, examples, tool defs), variable
  content last (user data, the question). This also maximizes prompt caching.
- Put the actual question near the bottom.
- Keep it lean: 150–300 words for most tasks. Reasoning degrades past ~3,000
  tokens. If a prompt is over 300 words, every sentence must earn its place.
- Positive framing over negation: "only use real data" beats "don't use mock
  data" (pink elephant problem).
- Avoid aggressive emphasis ("CRITICAL!", "YOU MUST", "NEVER EVER") — it
  degrades modern models, especially Claude.
</structure_rules>

## Fix workflow

<diagnose>
Run the failing prompt through these checks in order. Fix the first one that
applies; re-test before moving on. Change one thing at a time.

| Symptom | Likely cause | Fix |
|---|---|---|
| Vague, generic output | No context or audience | Add role/context; define the audience precisely |
| Wrong format | No format spec | Specify exact structure; add one example showing it |
| Missing parts of request | Too many stacked instructions | Cut to 3–5 constraints; split into staged prompts |
| Wrong tone/voice | No tone spec | Describe tone via writing choices, not labels ("friendly") |
| Hallucinated facts | No grounding rule | Add uncertainty rule + "label assumptions" + source constraints |
| Inconsistent output | No examples | Add 3–5 diverse few-shot examples in XML tags |
| Ignores mid-prompt rules | Info lost in the middle | Move critical rules to start/end |
| Confidence issues | Ambiguity | Define what "good" looks like explicitly |
| Overthink/verbose | Over-prompted | Trim instructions; audit anything over 300 words |
</diagnose>

Iteration is the method: run → identify the gap → add only what fixes that
specific gap → re-run. Never rewrite a working prompt wholesale.

## Model-specific tactics

| Model family | Style | Do | Avoid |
|---|---|---|---|
| **Claude** | Calm, explicit | XML tags (`<input>`, `<example>`), 3–5 examples, roles for creative tasks | Aggressive emphasis; implicit intent |
| **GPT-5.x** | Conversational, lean | Zero-shot first; pin production to model snapshots; use `reasoning_effort`/`verbosity` params | Explicit "think step by step" (router reasons internally) |
| **Gemini** | Short, direct | Always few-shot; question at the end; consistent example formatting | Long prompts; zero-shot reliance |

## Checklist

Run this before delivering any prompt:

- [ ] Outcome stated up front (what "good" looks like)
- [ ] Context only where it changes the answer
- [ ] 3–5 hard, testable constraints, not vague preferences
- [ ] Output format explicitly specified
- [ ] Examples (if needed) are diverse and XML-tagged
- [ ] Uncertainty + assumption-labeling rule present
- [ ] Critical rules at start/end, question near bottom
- [ ] Under 300 words unless genuinely necessary
- [ ] Positive framing used; no all-caps emphasis
- [ ] Model-specific tactics matched to the target model
