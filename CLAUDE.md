# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Master's Thesis (LaTeX) — Yimin Hu, KIT / SFB-1574 Circular Factory.
Topic: CoG-based geometric constraints + custom fingertip for reliable industrial deployment of vMF-Contact grasping on UR10e.

## Build Commands

```bash
# Full build with bibliography
pdflatex main.tex && bibtex main && pdflatex main.tex && pdflatex main.tex

# Quick draft compile (no bibliography update)
pdflatex main.tex
```

The system is missing `microtype` and `setspace` LaTeX packages. Fix with:
```bash
sudo apt-get install texlive-latex-extra
```

## File Structure

| File | Purpose |
|------|---------|
| `main.tex` | Full thesis document (single file, ~10,000 words target) |
| `references.bib` | IEEE BibTeX bibliography — 8 entries, all DOI-verified as of 2026-06-18 |
| `thesis-outline.md` | Authoritative chapter-by-chapter outline with word targets, evidence map, and open items |

## Thesis Architecture

Six-chapter IMRaD engineering variant:

1. **Introduction** — circular factory context → two algorithmic gaps (Gap A: no CoG prior, Gap B: no deployment evaluation)
2. **Related Work** — three-act evolution: ICP → learning-based → vMF-Contact
3. **System Design** — Contribution A (CoG constraints), B (custom fingertip), C (ROS2+Docker pipeline)
4. **Results** — 5 trials × 8 orientations × 9 positions; three conditions: Baseline 1 (<5%), Baseline 2 (~50%), Final (80%)
5. **Discussion** — benchmark dialogue (vs. Shi et al. 2025, 72.7%), limitations, future work
6. **Conclusion** — contributions summary, meta-implication

## Key Terminology

- **"geometric centroid"** not "centre of mass" — the CoG approximation (`cog_mean = 0.2×obb_center + 0.8×cog_pcd`) is geometry-based, not mass-based
- **Failure taxonomy** F1 (grip force), F2 (no valid pose), F3 (kinematic infeasibility), F4 (orientation error) — use exactly these labels
- **Benchmark comparison**: Final (80%) exceeds Shi et al. 72.7% despite three systematic disadvantages (lighter objects, stronger gripper, isolated metric in the benchmark); do not overclaim the +7.3 pp margin

## Pending Before Submission

See the `% PENDING` block at the top of `main.tex`:
- Fill Tables 4.1–4.3 from experimental logs
- Substitute chi-squared statistics in §4.2.1
- Verify 72.7% figure against ICRA 2025 proceedings PDF
- Enter total end-to-end cycle time in §3.6
- Verify full author lists for `cai2024` and `shi2025` in `references.bib`
- Insert Figures 3.1, 3.2, 3.3 (pipeline diagram, code snippet, CAD drawing)
- Add abstract (`/ars-abstract`) and AI-disclosure (`/ars-disclosure`)

## Citation Rules

- Format: IEEE; managed by `\usepackage{cite}` + BibTeX
- Two bib entries still have `author = {... and others}` placeholders — complete before final submission
- `besl1992` is flagged as a seminal >10-year-old source — acknowledge age when citing

## Key Repositories (External)

- Main pipeline: `github.com/SFB-Circular-Factory-WG-C/Yimin_Dochub`
- Algorithm fork: `github.com/YiminHu26/vMF-Contact` (branch: `dev_2`)
- Docker pipeline: `github.com/SFB-Circular-Factory-WG-C/docker_vmf_ur10e`
- Base paper: `arxiv.org/abs/2411.03591` (Shi et al., ICRA 2025)
