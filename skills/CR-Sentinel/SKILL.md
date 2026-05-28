---
name: CR-Sentinel
description: Final sanity-check pass for a machine learning paper before submission or at the camera-ready stage. Use this skill whenever the user is preparing a venue submission or camera-ready, mentions "camera ready", "CR check", "final submission", "pre-submission review", asks to verify numbers/citations/figures/references in a paper, or shares a LaTeX project / compiled PDF and wants a last look before submitting. Also trigger when the user mentions integrating rebuttal material, adding appendix experiments, or polishing an accepted paper for the proceedings — even if they don't explicitly say "camera ready". The skill ALWAYS first asks which venue + year and which stage (submission vs CR), then looks up that year's policy online before running checks. It catches trust-damaging issues (number mismatches, broken refs, unsupported claims, rebuttal-sounding text, forgotten de-anonymization or anonymity leaks) but does NOT rewrite the paper.
---

# CR-Sentinel — Camera-Ready Sanity Check

You are doing a **final pass** on an accepted ML paper. The author has limited time and a hard deadline. Your job is to surface issues that would embarrass the paper, break the build, or undermine reviewer trust — not to rewrite prose or relitigate the contribution.

## Core principle

Camera-ready edits should **clarify and strengthen** the accepted paper. New material (often pulled in from the rebuttal) must read like integrated validation, not pasted reviewer-response text. If a change alters the essential contribution or the headline claims, flag it loudly — it likely shouldn't be in a camera-ready.

## Before you start (mandatory)

Do not skip this step, even if the user dives straight into a file. Different venues have very different rules (page limit, required sections, anonymity policy, supplementary policy, font/template), and they change year-over-year. Running the generic checklist without knowing the venue produces wrong advice.

1. **Ask the user two things, briefly:**
   - **Which venue and year?** (e.g., NeurIPS 2026, ICLR 2026, CVPR 2026, ACL 2026, ICML 2026, EMNLP 2026, SIGGRAPH 2026.) If the user already mentioned it, confirm rather than re-ask.
   - **Which stage?** Submission (still double-blind, before reviews) or camera-ready (accepted, de-anonymizing and integrating rebuttal material). The checklist branches significantly between the two.

2. **Look up that year's policy on the web** before running checks. Use the official call-for-papers / author-instructions page for the specified venue+year. What to extract:
   - Page limit (main text vs. references vs. appendix — venues differ on whether each counts)
   - Anonymity policy (does it allow arXiv? what about acknowledgments and code links?)
   - Required sections (reproducibility checklist, broader / societal impact, ethics, limitations, author contributions)
   - Template / document class (the venue's CR class is often different from the submission class)
   - Supplementary material rules (separate upload, size limit, what's allowed)
   - Anything venue-specific (e.g., NeurIPS Datasets & Benchmarks track has its own checklist; ACL requires a Limitations section)

3. **Adjust the checklist to the policy you just looked up.** If the venue forbids X, don't flag X's absence; if the venue requires Y, make Y a blocker if missing. Cite the policy URL in your report so the author can verify.

4. **If you genuinely cannot reach the web** (offline, tool unavailable), say so explicitly and proceed with the generic checklist, but mark "policy-dependent" items in your report as **needs venue confirmation** rather than as pass/fail.

For the **submission stage**, drop the de-anonymization checks (§2) — anonymity is still in force — and instead verify that anonymity is **intact**: no real author names, no non-anonymous code/data URLs, no self-citations in the wrong form, no PDF metadata leakage. Keep the rest of the checklist mostly as-is, replacing "rebuttal residue" concerns with general draft-residue concerns.

## How to operate

1. **Locate the artifacts.** Ask for or find: the LaTeX source tree, the compiled PDF, the `.log` / `.blg` files if available, and (if relevant) the accepted/submitted version for diffing. If only the PDF is available, you can still do most checks via `pdftotext` (text), `pdfimages -list` (figures rendered), `pdffonts` (fonts embedded), and `pdfinfo` (metadata) — note what you couldn't verify.

2. **Diff against the accepted version when possible.** A camera-ready review is fundamentally about *what changed*. If you have both versions, run `latexdiff old.tex new.tex > diff.tex && pdflatex diff.tex` for a colored side-by-side, or `diff <(pdftotext old.pdf -) <(pdftotext new.pdf -)` for a quick text diff. Focus on the diff: new paragraphs, changed tables, new figures, new citations, modified abstract/intro/conclusion. These are where the risk lives.

3. **Work through the checklist below in order.** Some checks (build sanity, de-anonymization, broken refs) are cheap and catastrophic if missed — do those first. Others (tone, framing) need more judgment.

4. **Report as a prioritized punch list**, not prose. Use the structure in the "Reporting" section. Group by severity, cite file:line where you can, and quote the offending text so the author doesn't have to hunt.

5. **Do not edit the paper unless the user asks.** Your default output is a report. If they ask you to fix something, fix only what they pointed at — don't take the opportunity to "improve" surrounding text.

## Output language

Match the language of the user's conversation: if they're writing to you in Chinese, write the report headers, severity tags, and your own observations in Chinese; if English, English. **Always quote source text from the paper verbatim** — never translate a `\cite` key, a caption, a metric name, or a sentence from the paper. Code, file paths, command snippets, and LaTeX commands stay as-is regardless of language. If the user explicitly asks for a specific language ("用中文回答" / "respond in English"), honor that over the auto-detection.

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

Also check for **draft / template residue** that shouldn't be in a CR build:

- `\documentclass[...]{...}` — many venues use a separate CR class (e.g., `neurips_2024` → `neurips_2024_camera_ready`). Verify the class matches the venue's CR template.
- `\usepackage{draftwatermark}` or any watermark left on
- `\linenumbers` left enabled
- `\documentclass[draft]{...}` option still present
- Commented-out figures / tables sitting in the source from earlier revisions

A useful sweep:

```bash
grep -rn -E '(\?\?|\[\?\]|TODO|FIXME|XXX|draftwatermark|\\linenumbers)' --include='*.tex'
grep -nE '(undefined|multiply defined|There were undefined)' *.log *.blg 2>/dev/null
```

Every hit is a hard blocker. Report file:line and the surrounding snippet.

### 2. De-anonymization (CR-specific — easy to forget, catastrophic if missed)

The submitted version was double-blind; the CR version reveals identities for the first time. The transition is where the most embarrassing CR mistakes happen.

Check for **leftover anonymization**:

- Literal `Anonymous Submission`, `Anonymous Authors`, `Author Names Withheld` anywhere in source or rendered PDF
- `\anonymize{...}` macros / `\ifanonymous` blocks still active
- Bib entries like `author = {Anonymous}`, `author = {Author Et Al.}`, `[Authors, 2023]` placeholders for own prior work
- Anonymized URLs: `github.com/anonymous-author/...`, `anonymous.4open.science/...`, anonymized OpenReview links
- Self-citations that were neutralized (e.g., "as shown in prior work [42]" where [42] should now read normally)
- Footnote like "* Equal contribution. Author order randomized." that depended on the anonymity policy

Check **PDF metadata** (often forgotten):

```bash
pdfinfo paper.pdf | grep -E 'Title|Author|Subject|Creator'
```

The Title/Author fields should be the real ones, not "Anonymous Submission" or the template default.

Check **acknowledgments did not leak into the submitted version** (rare but happens): if the submitted PDF accidentally contained acknowledgments, that's a venue-policy violation worth noting.

If the venue allows arXiv preprints, confirm the **arXiv version policy** for CR: some venues require uploading a specific version, some require a delay. Flag if the author seems to be planning something incompatible.

### 3. Scope and claims

- Does the core method match the accepted version? If the method was renamed, restructured, or had a component swapped, flag it.
- Are there **new** claims in the abstract, intro, or conclusion that weren't in the accepted version? New claims need to be supported by evidence already in the paper.
- New experiments from the rebuttal should be framed as **additional validation** of existing claims — not as the new headline result.
- Abstract ↔ Intro ↔ Experiments ↔ Conclusion should tell a single consistent story. Spot-check that the headline numbers and verbs (e.g., "X× speedup", "matches quality of Y") agree across all four.

### 4. Numbers and tables

This is the highest-trust-cost category. A wrong abstract number is the kind of thing that gets screenshotted.

- **Recompute every headline number.** Speedups, latency reductions, FID/CLIP/LPIPS deltas, parameter counts. If the abstract says "2.3× faster", find the table cell that supports it and verify the arithmetic.
- **Abstract/intro numbers must match the final tables exactly.** Numbers often drift when tables get re-run for the camera-ready.
- **Best/second-best highlighting**: verify bold = actual best, underline = actual second-best, per column. Re-runs frequently invalidate the old bolding.
- **Metric direction and scale**: e.g., LPIPS (lower better), CLIP-I (higher better, 0–1 or 0–100?), percentages vs. ratios, ms vs. s. A 0.23 vs. 23% mismatch is easy to miss.
- **Table captions** must state: units, dataset/setting, baselines, and what speedups are *relative to*. "2.3×" against what?
- **Error bars / multiple runs**: if the submitted version reported ± std or "averaged over N seeds", every new row added during CR should do the same. Inconsistency reads as cherry-picking.
- Watch for tables that silently changed which baseline is the reference for "×" comparisons.
- Cross-check: if the same experiment appears in both the main text and the appendix, the numbers must match exactly.

### 5. Figures and captions

- Every figure renders in the compiled PDF (no missing-image placeholders, no rasterization disasters).
- Caption matches the figure: if caption says "left: X, right: Y", verify left actually shows X.
- Figure references in the text point to the figure that supports the sentence (`\ref{fig:foo}` resolves and is the right one).
- No hard-coded "Figure 3" in prose — should be `\cref{fig:label}` or `\ref{fig:label}`. Hard-coded numbers go stale when figures reorder.
- **B&W / colorblind readability**: many reviewers and reproducibility checkers still print papers. If a key figure relies on red vs. green to distinguish lines, flag it.

### 6. Math notation and equation references

- Same symbol means the same thing throughout: don't switch $\theta$ → $\phi$ for the parameter set between §3 and §5 without note.
- Vectors / matrices use a consistent style (bold vs. arrow vs. plain). Mixing is the most common notation drift.
- Equation references use `\eqref{eq:foo}` / `\cref{eq:foo}`, not hard-coded "Equation 4". Same for `\cref{thm:main}`, `\cref{lem:...}`, `\cref{alg:...}`.
- Algorithms / pseudocode: line numbers referenced from the text actually exist; variables in pseudocode are introduced before use.
- Theorem / lemma / proposition numbering is consistent (often controlled by `\numberwithin{theorem}{section}` — a class swap can break this silently).

### 7. References, citations, and URLs

- Every new citation added during the rebuttal points to a real, verifiable source. Be especially suspicious of model/paper names that sound plausible but you can't confirm — invented bib entries are a known failure mode.
- Cite **official model cards or project pages** for model references, not random blog posts.
- GitHub repos are *reference implementations*, not papers — don't cite a repo where a paper is expected.
- Company authors need braces in the .bib so BibTeX doesn't split them: `author = {{Black Forest Labs}}`. Check for "Labs, B. F." style breakage in the rendered bibliography.
- Skim the bibliography for duplicate entries (same paper, two keys) and for entries with missing venue/year.
- **Bib style consistency**: pick one — all entries use `NeurIPS` or all use `Advances in Neural Information Processing Systems`. Same for title-case vs. sentence-case.
- **URL / DOI sanity**: spot-check that DOIs resolve (no 404s) and `\url{}` entries don't break across lines because of unescaped `_` or `%`. Replace any anonymized review URLs (`openreview.net/forum?id=...` from the submission portal) with their final non-anonymous form if the venue requires it.
- **Code / data release**: if the paper points to a code repo or dataset, the link should be a **stable release / tag**, not a `main` branch or a temporary zip. A README in that repo should mention the paper.

### 8. Appendix integration

- Every `\ref{app:...}` in the main text resolves to an appendix that actually exists.
- Appendix prose should sound like paper prose — not "We thank Reviewer 2 for pointing out...". Rebuttal voice is the #1 tell that material was pasted in.
- Heavy material (pseudocode, extra baselines, qualitative grids, derivations) belongs in the appendix. If something heavy is in the main text only because it was a rebuttal response, consider whether it should move.
- If the appendix supports a load-bearing claim, the main text should mention the takeaway and point to the appendix — don't bury key validation.

### 9. Experimental setup details

Verify the following are stated and internally consistent. Use these as a starting list and adapt to the paper's domain (image generation, NLP, RL, etc.):

- Model names and exact versions (e.g., `FLUX.1-dev` vs `FLUX.2-Klein`, `Llama-3.1-8B-Instruct` vs `Llama-3.2-8B` — version digits matter).
- Dataset names and splits, including any subset selection.
- Domain-specific hyperparameters: resolution / denoising steps / sampler (for diffusion), context length / max tokens (for LMs), episode budget (for RL), LoRA rank / trainable params / learning rate / batch size — whatever the contribution rests on.
- Hardware: GPU type and count, GPU-hours or wall-clock for training.
- **Unique items vs. training exposures** — in image-gen this is "unique images vs. image-level exposures"; in LM this is "unique tokens vs. tokens seen". These are different and frequently conflated; if the paper reports one, make sure it doesn't also imply the other.
- Baseline training settings: are baselines trained on the same budget? If not, the comparison needs a caveat.
- Random seeds / number of independent runs — if claimed, confirmed across all rows of all tables.

### 10. Language and tone

- Search for rebuttal residue: "we thank the reviewer", "in response to", "as the reviewer noted", "addressing the concern", "Reviewer 2", "the rebuttal", "during rebuttal". Any hit is a blocker — rewrite in neutral voice.
- Avoid absolute verbs: "proves", "fails", "obviously", "clearly demonstrates". Prefer hedged, scoped claims: "on FLUX.2-Klein at 1024² with 28 steps, our method achieves ...".
- Claims should be tied to a specific setting, not stated as universal truths.
- Spelling pass on **new material only** — the rebuttal-added paragraphs are where typos cluster, not the original body.

### 11. Acknowledgments and author-info first appearance

The acknowledgments section often appears for the first time in the CR version. It's a small piece of text with an outsized number of failure modes.

- Acknowledged people / labs actually consented to be acknowledged.
- Funding sources use the exact format the venue / funder requires (e.g., NSF grant numbers as `NSF-XXXXXXX`, ERC grant IDs).
- No conflicts of interest leak in (e.g., acknowledging a reviewer or area chair you happen to know).
- Acknowledgments don't push the paper over the page limit.
- Author list, ordering, affiliations, and ORCIDs in the PDF match what's registered in the venue's CR portal.
- Equal-contribution markers (`*`, `†`) are defined in a footnote and consistent with what coauthors agreed to.
- Corresponding author email is a stable, monitored address (not a soon-expiring student account if avoidable).

### 12. Venue-specific checklist, ethics, and broader impact

Many venues (NeurIPS, ICLR, ACL, EMNLP, ICML) require specific extra sections or forms in the CR version. Check the **venue's CR instructions page** if available; otherwise check for these common requirements:

- **Reproducibility / paper checklist**: if the submitted version had checklist answers, the CR version's answers must be **updated** to reflect any new experiments, datasets, or claims added during the rebuttal. A "Yes — see §4.3" that points to a now-renumbered section is a blocker.
- **Broader impact / societal impact statement**: required by NeurIPS and some others. If the rebuttal added new applications, the broader-impact section should reflect them.
- **Ethics statement**: if the paper uses human subjects, sensitive data, or has dual-use risks, the venue likely requires a statement. Confirm it's present and current.
- **Limitations section**: ICLR and others now require explicit limitations; check it exists and hasn't been silently weakened from the accepted version.
- **Data / code release statement**: many venues now require a yes/no with link or justification.

**Common venue-specific reminders to actively surface to the user** (don't wait for them to ask — these are silently fatal):

- **NeurIPS** — the *Paper Checklist* is mandatory and lives at the end of the paper (after references, before appendix; does not count toward page limit). It must be filled in with `\answerYes{...}`, `\answerNo{...}`, `\answerNA{...}` for every item, and every answer needs a justification or a section pointer. If the rebuttal added new experiments, datasets, or claims, the corresponding checklist answers must be re-checked. **Explicitly remind the user to fill / refresh the checklist** if you see the template still has unfilled `\answerTODO{}` entries or if the checklist seems to reference a now-renumbered section.
- **NeurIPS Datasets & Benchmarks track** — has its own additional checklist; don't confuse it with the main-track checklist.
- **ICLR** — requires a Reproducibility Statement and a Limitations / Ethics statement; check both exist and are tied to actual sections.
- **ACL / EMNLP / NAACL** — require an explicit Limitations section (not optional, not a paragraph in the conclusion). Also have a Responsible NLP Research checklist on submission.
- **ICML** — Broader Impact statement is required.
- **CVPR / ICCV / ECCV** — typically no required checklist, but page-limit rules around references and supplementary differ — verify against the year's CFP.

Whatever the venue, if a required checklist or statement is missing or stale, **list it as a Blocker, not a soft reminder**.

### 13. Final submission-process risks

These aren't in the PDF — ask the author:

- Page limit (including/excluding references and appendix per the venue's rules).
- Required forms (camera-ready agreement, copyright, ethics statement).
- Coauthor sign-off on the final version.
- Author list, affiliations, ORCID, and ordering match the venue's CR portal.
- Funding acknowledgments and conflicts-of-interest statements present if required.
- Supplementary material (code, video) policy: separate upload? Frozen anonymized link replaced with a real one?
- **PDF font embedding**: `pdffonts paper.pdf` — every row's `emb` column should be `yes`. IEEE / ACM CR portals reject PDFs with un-embedded fonts.
- **PDF version**: some venues require PDF 1.5+ or PDF/A. Check `pdfinfo paper.pdf | grep "PDF version"` if the venue specifies one.

## Reporting

Default output format — adapt depth to what was actually found:

```
# CR-Sentinel Report — <paper title or filename>

**Venue / year / stage**: NeurIPS 2026 — camera-ready
**Policy source**: https://neurips.cc/Conferences/2026/PaperInformation/CameraReady
**Key policy facts applied**: 9-page main text + unlimited appendix; Limitations section required; Broader Impact required; CR class `neurips_2026_camera_ready`.

## Blockers (must fix before submission)
- [build] main.tex:412 — unresolved \ref{fig:teaser-v2} → renders as "Figure ??"
- [deanon] sec_intro.tex:28 — "Anonymous Submission" still in title block
- [numbers] abstract.tex:8 — claims "2.4× speedup" but Table 2 (row "Ours, 1024²") supports 2.1×

## Should-fix (trust / consistency)
- [tone] sec_appendix.tex:33 — "We thank Reviewer 2 for ..." — reads as rebuttal
- [citation] new entry `flux2klein2026` — could not verify; please confirm against official model card
- [url] refs.bib:142 — DOI returns 404; double-check the entry

## Worth a look (judgment calls)
- [framing] intro paragraph 3 introduces "memory-efficient training" as a contribution; this wasn't in the accepted abstract
- [acks] sec_acks.tex:5 — funding line format may need to match NSF grant style

## Could not verify
- LoRA rank stated as 64 in §4.1 but no training-config table; ask the author to confirm
- No accepted-version PDF provided, so diff-based checks were skipped

## Sanity passed
- All `\ref` and `\cite` resolve in the compiled PDF
- PDF metadata Title/Author look correct (`pdfinfo`)
- Bibliography has no duplicate keys
- No `\linenumbers` / draftwatermark residue

## Submission-process checklist (asks for the author)
- [ ] Page count under venue limit?
- [ ] Camera-ready agreement signed?
- [ ] Coauthors approved final?
- [ ] All PDF fonts embedded (`pdffonts`)?
- [ ] Reproducibility checklist updated for rebuttal-added experiments?
- [ ] Code/data link is a stable release, not `main`?

## Section status (at-a-glance)

| § | Section                                  | Status | Open issues |
|---|------------------------------------------|--------|-------------|
| 1 | Build sanity & unresolved refs           | ✗      | 1 blocker (unresolved \ref) |
| 2 | De-anonymization                         | ✗      | 1 blocker ("Anonymous Submission" in title) |
| 3 | Scope and claims                         | ⚠      | 1 to review (new contribution in intro) |
| 4 | Numbers and tables                       | ✗      | 1 blocker (abstract ≠ Table 2) |
| 5 | Figures and captions                     | ✓      | clean |
| 6 | Math notation and equation references    | ✓      | clean |
| 7 | References, citations, and URLs          | ⚠      | 1 to verify (DOI 404), 1 unverifiable citation |
| 8 | Appendix integration                     | ⚠      | 1 rebuttal-voice paragraph |
| 9 | Experimental setup details               | ?      | could not verify (no config table) |
|10 | Language and tone                        | ⚠      | rebuttal residue in appendix |
|11 | Acknowledgments & author info            | ⚠      | funding format may need adjusting |
|12 | Venue-specific (checklist / ethics)      | ✗      | NeurIPS Paper Checklist still has `\answerTODO{}` |
|13 | Final submission-process risks           | ?      | author confirmation needed |

**Legend**: ✓ clean — ⚠ should-fix or worth-a-look — ✗ blocker — ? could not verify

**Bottom line**: <one-sentence summary, e.g. "5 blockers must be fixed before submission; 4 should-fix items; 2 items need author confirmation.">
```

Keep each finding to one line + a short quote when useful. The author is on a deadline — they should be able to act on the report in one pass. Include a **Sanity passed** section so the author knows what's verified clean, not just what's broken.

The **Section status table** at the end is mandatory. It gives the author a single-glance map of "where am I in this review, what's still open, what's clean". When the user comes back after fixing issues, regenerate the table with updated statuses so they can see progress without re-reading the whole report. Mark items resolved (`✓`) only after re-verifying — don't trust the author's "fixed" without re-running the relevant check.

## What NOT to do

- Don't rewrite prose unless asked. A finding is enough; the author decides the fix.
- Don't propose new experiments. Camera-ready is too late.
- Don't relitigate the contribution or suggest restructuring the paper.
- Don't invent citations or "fix" a citation by substituting a different paper — flag and let the author resolve.
- Don't claim a check passed if you couldn't actually verify it (e.g., you didn't have the .log file). Put it under "Could not verify".
- Don't translate quoted source text. Your prose follows the conversation language; the paper's text is reproduced verbatim.

## Final rule

A strong camera-ready feels **reliable**: every number matches a table, every citation is real, every reference resolves, every figure says what it shows, every identity is correctly de-anonymized, and every new addition fits naturally into the accepted paper. Your job is to surface every place that isn't yet true.
