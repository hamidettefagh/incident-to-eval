# Incident to eval

A Claude skill that turns a production AI agent incident into a golden eval case.

An incident is an eval you already paid for. Most teams write the postmortem, fix the immediate cause, and throw the eval away. The failure comes back later wearing different clothes, because nothing runs on every change to make it unrepeatable. This skill does the step that gets skipped: it reconstructs the incident, classifies the failure against an agent-specific taxonomy, and writes a portable regression case that any runner can enforce.

## The loop it closes

The [design gate](https://github.com/hamidettefagh/agent-or-workflow) decides what to build. The [ship gate](https://github.com/hamidettefagh/agent-production-readiness) proves it is ready. When something gets through both anyway, this skill turns the failure into evidence the ship gate checks forever after. The write-up behind the method is at [hamidettefagh.com/two-gates](https://hamidettefagh.com/two-gates).

```
design gate -> build -> ship gate -> run -> incident
     ^                      ^                  |
     |                      +--- eval case <---+
     +------ revisit when the case changes the design
```

## What it is not

Not an eval platform and not a runner. Platforms already capture traces and datasets; the step nobody ships is the authoring judgment in between: deciding what the failure actually was and what a good assertion looks like. The output is a plain JSON case file that maps onto whatever you run today, a CI job, an eval platform, or a test file.

## The format is deterministic

Every case is validated by [`scripts/validate-eval.mjs`](scripts/validate-eval.mjs): pinned schema, closed assertion set, exact errors, stable exit codes. Same file in, same verdict out. The judgment lives in reading the incident; none of it lives in the format check. Golden tests in [`scripts/test.mjs`](scripts/test.mjs) keep the shipped example, the template, and every rejection path honest; run `node scripts/test.mjs`.

## Install

This follows the open [Agent Skills](https://code.claude.com/docs/en/skills) format, so it works anywhere skills are supported. To use it in Claude Code, put this directory in your skills folder:

```bash
git clone https://github.com/hamidettefagh/incident-to-eval.git \
  ~/.claude/skills/incident-to-eval
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to scope it to one repo. The validator needs Node; nothing else.

## Use

Hand Claude the failure, in whatever form you have it:

> Our support agent quoted last quarter's refund policy to a customer yesterday. Turn it into an eval.

> Here is the postmortem from Tuesday's incident. I want a regression test out of it.

The skill reconstructs what happened, classifies it against the failure taxonomy, writes the case, validates it, and tells you where the guard should run. You can also drive the validator directly:

```bash
node scripts/validate-eval.mjs --template          # starter case
node scripts/validate-eval.mjs the-case.json       # validate
node scripts/validate-eval.mjs --md the-case.json  # summary for the PR
```

## What a case looks like

One incident becomes one JSON document: the source incident, the classified failure, the smallest reproducing input, assertions from a closed set, and the guard that says when it runs. The full format is in [`references/eval-case-schema.md`](references/eval-case-schema.md), and the nine-category failure taxonomy, with the eval each mode owes you, is in [`references/failure-taxonomy.md`](references/failure-taxonomy.md).

The worked example is real: [`assets/example-airline-grounding.json`](assets/example-airline-grounding.json) is the go-live incident from the [airline case study](https://hamidettefagh.com/work/airline-super-agent), rendered as the eval that would have caught it on a Tuesday afternoon instead of a Friday night.

## Why

The most expensive part of a production failure is the information in it, and most teams spend that information on a document nobody re-reads. A golden case spends it on a guard that runs on every change. The score in a readiness review tells you evaluation is thin; this is the skill that thickens it, one real failure at a time.

---

Hamid Ettefagh, [hamidettefagh.com](https://hamidettefagh.com)
