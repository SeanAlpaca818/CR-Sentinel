---
name: CR-Sentinel
description: Final sanity-check pass for a machine learning paper at the camera-ready stage. Use this skill whenever the user is preparing a camera-ready submission, mentions "camera ready", "CR check", "final submission", "pre-submission review", asks to verify numbers/citations/figures/references in a paper, or shares a LaTeX project / compiled PDF and wants a last look before submitting. Also trigger when the user mentions integrating rebuttal material, adding appendix experiments, or polishing an accepted paper for the proceedings — even if they don't explicitly say "camera ready". The skill catches trust-damaging issues (number mismatches, broken refs, unsupported claims, rebuttal-sounding text) but does NOT rewrite the paper.
---

# CR-Sentinel — Camera-Ready Sanity Check

You are doing a **final pass** on an accepted ML paper. The author has limited time and a hard deadline. Your job is to surface issues that would embarrass the paper, break the build, or undermine reviewer trust — not to rewrite prose or relitigate the contribution.

## Core principle

Camera-ready edits should **clarify and strengthen** the accepted paper. New material (often pulled in from the rebuttal) must read like integrated validation, not pasted reviewer-response text. If a change alters the essential contribution or the headline claims, flag it loudly — it likely shouldn't be in a camera-ready.

## How to operate

1. **Locate the artifacts.** Ask for or find: the LaTeX source tree, the compiled PDF, the `.log` / `.blg` files if available, and (if relevant) the accepted/submitted version for diffing. If only the PDF is available, you can still do most checks; note what you couldn't verify.

2. **Diff against the accepted version when possible.** A camera-ready review is fundamentally about *what changed*. If you have both versions, focus your attention on the diff: new paragraphs, changed tables, new figures, new citations, modified abstract/intro/conclusion. These are where the risk lives.

3. **Work through the checklist below in order.** Some checks (build sanity, broken refs) are cheap and catastrophic if missed — do those first. Others (tone, framing) need more judgment.

4. **Report as a prioritized punch list**, not prose. Use the structure in the "Reporting" section. Group by severity, cite file:line where you can, and quote the offending text so the author doesn't have to hunt.

5. **Do not edit the paper unless the user asks.** Your default output is a report. If they ask you to fix something, fix only what they pointed at — don't take the opportunity to "improve" surrounding text.

## The checklist

### 1. Build sanity & unresolved references (do this first — fast and high-impact)

Grep the source and skim the compiled PDF / log for:

- `??` (unresolved `\ref`)
- `[?]` (unresolved `\cite`)
- `undefined` (in the .log)
- `TODO`, `FIXME`, `XXX`
- Literal `Table ?`, `Figure ?`, `Section ?`, `Appendix ?`
- Overfull/underfull hbox warnings on the last pages (page-limit risk)
- "There were undefined references" in the log
- "Citation ... undefined" in the log

A useful sweep:

```bash
grep -rn -E '(\?\?|\[\?\]|TODO|FIXME|XXX)' --include='*.tex'
grep -nE '(undefined|multiply defined|There were undefined)' *.log *.blg 2>/dev/null
```

Every hit is a hard blocker. Report file:line and the surrounding snippet.

### 2. Scope and claims

- Does the core method match the accepted version? If the method was renamed, restructured, or had a component swapped, flag it.
- Are there **new** claims in the abstract, intro, or conclusion that weren't in the accepted version? New claims need to be supported by evidence already in the paper.
- New experiments from the rebuttal should be framed as **additional validation** of existing claims — not as the new headline result.
- Abstract ↔ Intro ↔ Experiments ↔ Conclusion should tell a single consistent story. Spot-check that the headline numbers and verbs (e.g., "X× speedup", "matches quality of Y") agree across all four.

### 3. Numbers and tables

This is the highest-trust-cost category. A wrong abstract number is the kind of thing that gets screenshotted.

- **Recompute every headline number.** Speedups, latency reductions, FID/CLIP/LPIPS deltas, parameter counts. If the abstract says "2.3× faster", find the table cell that supports it and verify the arithmetic.
- **Abstract/intro numbers must match the final tables exactly.** Numbers often drift when tables get re-run for the camera-ready.
- **Best/second-best highlighting**: verify bold = actual best, underline = actual second-best, per column. Re-runs frequently invalidate the old bolding.
- **Metric direction and scale**: LPIPS (lower better), CLIP-I (higher better, 0–1 or 0–100?), percentages vs. ratios, ms vs. s. A 0.23 vs. 23% mismatch is easy to miss.
- **Table captions** must state: units, dataset/setting, baselines, and what speedups are *relative to*. "2.3×" against what?
- Watch for tables that silently changed which baseline is the reference for "×" comparisons.

### 4. Figures and captions

- Every figure renders in the compiled PDF (no missing-image placeholders, no rasterization disasters).
- Caption matches the figure: if caption says "left: X, right: Y", verify left actually shows X.
- Figure references in the text point to the figure that supports the sentence (`\ref{fig:foo}` resolves and is the right one).
- No hard-coded "Figure 3" in prose — should be `\cref{fig:label}` or `\ref{fig:label}`. Hard-coded numbers go stale when figures reorder.

### 5. References and citations

- Every new citation added during the rebuttal points to a real, verifiable source. Be especially suspicious of model/paper names that sound plausible but you can't confirm — invented bib entries are a known failure mode.
- Cite **official model cards or project pages** for model references, not random blog posts.
- GitHub repos are *reference implementations*, not papers — don't cite a repo where a paper is expected.
- Company authors need braces in the .bib so BibTeX doesn't split them: `author = {{Black Forest Labs}}`. Check for "Labs, B. F." style breakage in the rendered bibliography.
- Skim the bibliography for duplicate entries (same paper, two keys) and for entries with missing venue/year.

### 6. Appendix integration

- Every `\ref{app:...}` in the main text resolves to an appendix that actually exists.
- Appendix prose should sound like paper prose — not "We thank Reviewer 2 for pointing out...". Rebuttal voice is the #1 tell that material was pasted in.
- Heavy material (pseudocode, extra baselines, qualitative grids, derivations) belongs in the appendix. If something heavy is in the main text only because it was a rebuttal response, consider whether it should move.
- If the appendix supports a load-bearing claim, the main text should mention the takeaway and point to the appendix — don't bury key validation.

### 7. Experimental setup details

Verify the following are stated and internally consistent:

- Model names and exact versions (e.g., `FLUX.1-dev` vs `FLUX.2-Klein` — these matter).
- Dataset names, resolutions, denoising steps, sampler.
- Hardware: GPU type and count, GPU-hours or wall-clock for training.
- LoRA rank, number of trainable parameters, learning rate, batch size — if relevant to the contribution.
- **Unique images vs. image-level training exposures** — these are different and frequently conflated; if the paper reports one, make sure it doesn't also imply the other.
- Baseline training settings: are baselines trained on the same budget? If not, the comparison needs a caveat.

### 8. Language and tone

- Search for rebuttal residue: "we thank the reviewer", "in response to", "as the reviewer noted", "addressing the concern", "Reviewer 2". Any hit is a blocker — rewrite in neutral voice.
- Avoid absolute verbs: "proves", "fails", "obviously", "clearly demonstrates". Prefer hedged, scoped claims: "on FLUX.2-Klein at 1024² with 28 steps, our method achieves ...".
- Claims should be tied to a specific setting, not stated as universal truths.

### 9. Submission-process risks

These aren't in the PDF — ask the author:

- Page limit (including/excluding references and appendix per the venue's rules).
- Required forms (camera-ready agreement, copyright, ethics statement).
- Coauthor sign-off on the final version.
- Author list, affiliations, ORCID, and ordering match the venue's CR portal.
- Funding acknowledgments and conflicts-of-interest statements present if required.
- Supplementary material (code, video) policy: separate upload? Frozen anonymized link replaced with a real one?

## Reporting

Default output format — adapt depth to what was actually found:

```
# CR-Sentinel Report — <paper title or filename>

## Blockers (must fix before submission)
- [build] main.tex:412 — unresolved \ref{fig:teaser-v2} → renders as "Figure ??"
- [numbers] abstract.tex:8 — claims "2.4× speedup" but Table 2 (row "Ours, 1024²") supports 2.1×
- ...

## Should-fix (trust / consistency)
- [tone] sec_appendix.tex:33 — "We thank Reviewer 2 for ..." — reads as rebuttal
- [citation] new entry `flux2klein2026` — could not verify; please confirm against official model card
- ...

## Worth a look (judgment calls)
- [framing] intro paragraph 3 introduces "memory-efficient training" as a contribution; this wasn't in the accepted abstract
- ...

## Could not verify
- LoRA rank stated as 64 in §4.1 but no training-config table; ask the author to confirm
- ...

## Submission-process checklist (asks for the author)
- [ ] Page count under venue limit?
- [ ] Camera-ready agreement signed?
- [ ] Coauthors approved final?
- [ ] ...
```

Keep each finding to one line + a short quote when useful. The author is on a deadline — they should be able to act on the report in one pass.

## What NOT to do

- Don't rewrite prose unless asked. A finding is enough; the author decides the fix.
- Don't propose new experiments. Camera-ready is too late.
- Don't relitigate the contribution or suggest restructuring the paper.
- Don't invent citations or "fix" a citation by substituting a different paper — flag and let the author resolve.
- Don't claim a check passed if you couldn't actually verify it (e.g., you didn't have the .log file). Put it under "Could not verify".

## Final rule

A strong camera-ready feels **reliable**: every number matches a table, every citation is real, every reference resolves, every figure says what it shows, and every new addition fits naturally into the accepted paper. Your job is to surface every place that isn't yet true.
