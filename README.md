# staged-plan-execution

A Trae SKILL that wraps any multi-step fix/improvement plan into a complete, agent-executable prompt — with built-in quality gates, atomic commits, CHANGELOG rules, and multi-session handoff.

[中文文档](README_CN.md)

## What it does

Give it a multi-step plan like this:

```markdown
## Step 1: add lock to foo
**File**: `src/foo.ts`
**Change**: import withManifestLock, wrap function body
...
## Step 2: desensitize bar error
...
| Step | Content | Files |
| 1 | lock foo | src/foo.ts |
| 2 | desensitize bar | src/bar.ts |
After each step: npm run typecheck && npm run lint && npm test
```

It produces a complete prompt like this:

```text
You need to execute the following revision plan for the current project,
completing each step in order. Run quality gates after each step before
proceeding to the next.

## Project style requirements (all steps)
(Points at the source files the plan touches + sibling test files as style examples)

## Atomic commit rules (strict)
(fix + docs commit split rules)

## CHANGELOG rules (strict)
(location, revision, date rules)

## README update rules
(conditional logic)

## Context management and stage handoff
(four-backtick handoff prompt — 7 mandatory sections + skeleton)

---

## <plan title>
(plan body preserved verbatim, project name → "current project",
code-block identifiers untouched)

---

## After all steps complete
(final verification checklist)
```

The generated prompt is designed for **multi-session relay execution**: after each step, the executor agent outputs a handoff prompt (wrapped in four backticks) that the user copies into a new session to continue.

## Why

When executing a multi-step plan across multiple agent sessions, three problems recur:

1. **Context drift** — later sessions forget conventions established in earlier ones
2. **Inconsistent handoffs** — each session invents its own "where was I" format
3. **Style regression** — agents skip quality gates or mix code + docs in one commit when context gets tight

This skill bakes the conventions into a template-driven generator, so every handoff carries the full execution framework forward verbatim.

## Structure

```
.trae/skills/staged-plan-execution/
├── SKILL.md                       # Entry: purpose, scope, generation rules, reference table
├── references/                    # Hard constraints, loaded verbatim into generated prompt
│   ├── project-style-guide.md     # 6 style rules + ecosystem identification + file-pointing
│   ├── atomic-commit-rules.md     # fix + docs commit split + build artifact scope
│   ├── changelog-rules.md         # Keep a Changelog conventions
│   ├── readme-rules.md            # Conditional README update logic
│   ├── handoff-format.md          # 7 mandatory handoff prompt sections + skeleton
│   └── final-verification.md      # Post-completion verification checklist
├── templates/
│   └── plan-prompt-template.md    # Overall skeleton of the generated prompt + placeholder rules
└── examples/
    └── sample-plan-to-prompt.md   # Worked example: plan input → generated prompt
```

The split is intentional:

- **SKILL.md is lean** — fast to load, lets the agent quickly decide whether to invoke this skill
- **references/ holds the heavy constraints** — loaded on demand, then copied verbatim into the generated prompt. This preserves reproducibility: the same plan always yields the same prompt, regardless of agent memory
- **templates/ defines assembly order** — the agent doesn't improvise structure
- **examples/ anchors understanding** — shows the input → output shape concretely

## Scope

Defaults assume this stack and these conventions:

- **Language**: TypeScript / npm (`npm run typecheck` / `lint` / `test` / `build`)
- **Testing**: `node:test` + `tsx`, test files named `xxx.test.ts`, temp-dir isolation
- **CHANGELOG**: Keep a Changelog style (`## [Unreleased]`)
- **Execution model**: multi-session relay — each step ends with a four-backtick handoff prompt

The skill is tuned for the **DeepSeek harness plugin ecosystem** (bilingual CONTRIBUTING, `src/` + `test/` + `lib/` layout, `dsh-` CLI prefix). For other TS/npm projects it still works; for non-TS projects, the `references/` would need adaptation.

## Key design choices

| Choice | Rationale |
|--------|-----------|
| Plan content is preserved verbatim | The plan author's intent must not be paraphrased or summarized |
| Project name → "current project" | Decouples the skill from any specific repo |
| Code-block identifiers stay as-is | They're part of the code, not prose |
| Agent must Read `references/` at generation time | Prevents memory drift; same plan → same prompt every time |
| Agent scans plan to point at specific files | Restores plan↔prompt tightness — the style-guide section names the exact files the plan touches and their sibling test files |
| Four-backtick handoff wrapper | Allows inner triple-backtick code blocks to survive intact when user copies the handoff |

## How to use

### As a Trae SKILL

1. Place the `staged-plan-execution/` directory under `.trae/skills/` in your workspace
2. Provide a multi-step plan to the agent
3. The agent recognizes the skill, reads `references/` and `templates/`, then outputs the complete prompt in the conversation
4. Copy the generated prompt into a fresh agent session to begin execution

### As a reference for other tooling

The `references/` and `templates/` are plain Markdown. You can adapt them into prompts for other agent frameworks, CI pipelines, or documentation generators — the conventions themselves are tool-agnostic.

## Limitations

- **TS/npm-centric**: `references/` uses JS-specific terms (JSDoc, `after()` hooks, `lib/` build artifacts). For Python/Go/Rust, adapt the style guide
- **Single-plan-per-invocation**: doesn't compose multiple plans into one prompt
- **No plan validation**: doesn't check whether the plan is sound — it wraps whatever the user provides

## License

MIT
