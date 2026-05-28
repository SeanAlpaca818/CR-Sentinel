# CR-Sentinel

A [Claude Code](https://claude.com/claude-code) skill for the **final sanity-check pass** on a machine learning paper at the camera-ready stage.

It does **not** rewrite your paper. It surfaces the kinds of issues that damage reviewer trust or break the submission:

- Unresolved `\ref` / `\cite` (`??`, `[?]`), undefined labels, leftover `TODO`/`FIXME`
- Abstract / intro numbers that don't match the final tables
- Best / second-best highlighting that went stale after a re-run
- Figure captions that don't match the figure
- Citations that look plausible but can't be verified
- Rebuttal residue ("we thank the reviewer", "in response to...") leaking into the camera-ready
- Appendix references that don't resolve
- Submission-process risks (page limit, forms, coauthor sign-off)

Output is a prioritized punch list: **Blockers / Should-fix / Worth a look / Could not verify / Submission checklist**.

---

## Install

### Option A — Plugin marketplace (recommended)

One-line install inside Claude Code:

```
/plugin marketplace add SeanAlpaca818/CR-Sentinel
/plugin install CR-Sentinel@CR-Sentinel
```

Restart Claude Code if needed. The skill becomes available globally.

### Option B — Manual install (plain skill)

Clone the skill directly into your Claude skills directory:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/SeanAlpaca818/CR-Sentinel.git /tmp/cr-sentinel
cp -r /tmp/cr-sentinel/skills/CR-Sentinel ~/.claude/skills/
```

Or as a single command:

```bash
git clone --depth 1 https://github.com/SeanAlpaca818/CR-Sentinel.git \
  && cp -r CR-Sentinel/skills/CR-Sentinel ~/.claude/skills/ \
  && rm -rf CR-Sentinel
```

Start a new Claude Code session and the skill will be picked up automatically.

---

## Usage

Once installed, trigger the skill in any of these ways:

- **Explicit**: `/CR-Sentinel` or "run CR-Sentinel on this paper"
- **Implicit** — the skill auto-triggers when you mention things like:
  - "I'm prepping the camera-ready for our NeurIPS paper, can you do a final check on `main.tex`?"
  - "Help me integrate the rebuttal experiments into the appendix without it sounding like a rebuttal."
  - "Verify the numbers in the abstract match Table 2."
  - "Final submission check — anything broken in the PDF?"

### What to give it

For the best report, point Claude at:

1. The LaTeX source tree (`main.tex` and friends)
2. The compiled PDF
3. The `.log` / `.blg` files if you have them
4. The accepted / originally-submitted version (so the skill can diff against it)

If you only have the PDF, the skill still runs — it just notes what it couldn't verify.

---

## What it checks

| Category | Examples |
|---|---|
| Build sanity | `??`, `[?]`, undefined refs, `TODO`/`FIXME`, overfull boxes near page limit |
| Scope & claims | New claims in abstract that weren't in the accepted version |
| Numbers & tables | Headline speedups, best/second-best bolding, metric scales, units in captions |
| Figures | Captions matching content, left/right descriptions, hard-coded figure numbers |
| Citations | Unverifiable entries, GitHub-as-paper, unbraced company authors (`{{Black Forest Labs}}`) |
| Appendix | Resolved `\ref{app:...}`, rebuttal-voice prose, key takeaways surfaced to main text |
| Setup details | Model versions, LoRA rank, GPU-hours, unique images vs. training exposures |
| Tone | Rebuttal residue, absolute verbs ("proves", "obviously"), unscoped claims |
| Submission risks | Page limit, required forms, coauthor sign-off, supplementary policy |

Full details in [`skills/CR-Sentinel/SKILL.md`](skills/CR-Sentinel/SKILL.md).

---

## License

MIT
