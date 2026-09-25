# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Course website + materials for **BSTA 551, Theory of Statistical Inference** (OHSU, Winter 2026), taught by Jessica Minnier. A Quarto website with R-executed code chunks — no application, no test suite, no package. "Building" means rendering Quarto; "shipping" means committing the rendered `docs/` output.

Repo: `github.com/jminnier/BSTA551_W26`. Assignments are submitted through Sakai; everything else lives on this site. The site is adapted from Dr. Nicky Wakim's BSTA 550 website, which explains several leftovers (see *Legacy content*).

**Course context.** An 11-week masters-level course in theoretical statistical inference: distributions of random variables (location-scale and exponential families), data reduction (sufficiency, completeness), estimation (MME, MLE), convergence and finite/large-sample properties, interval estimation, hypothesis testing, asymptotic tests (LRT, score, Wald), and simulation to evaluate methods. Students arrive with one quarter of probability (BSTA 550) and working R. Learning objectives: (1) explain major concepts and theorems in inference, (2) connect theory to statistical analyses, (3) conduct simulations to study and evaluate methods.

## Authoritative sources

When these disagree, the left column wins. Do not infer what the course covers from the file tree — drafts and leftovers live alongside taught material.

| Question | Read this | Do **not** trust |
|---|---|---|
| What was taught, in what order, with which textbook sections? | `schedule.qmd` | the `lessons/` directory listing, or `draft_schedule.qmd` (a BSTA **550 probability** schedule) |
| Which homework was actually assigned? | `homeworks.qmd` (HW 0–7) | `homework/` listing — `HW_08`–`HW_10` were never assigned, and `hw_answers/` is 550 |
| Scope, objectives, grading, policies, no-class dates | `syllabus.qmd` | `_course-settings.yml` (Fall 2023, unused); `quiz.qmd` (BSTA 551 gives no quizzes) |
| Every date in the term | `class_dates.R` | any date typed into a `.qmd` |
| What gets published | the `render:` list in `_quarto.yml` | the presence of a `.qmd` on disk |
| Textbook section and exercise numbers | the Devore PDF (see *Reference materials*) | slide citations — Lesson 18 miscites "DBC 9.5" for two-sample z/t |

**Taught content is exactly lessons 0–23 and HW 0–7.** Everything else on disk is a draft, a BSTA 550 leftover, or an unassigned extra, and is excluded from the render list.

---

## Authoring conventions

These apply to every lesson, homework, and solution file you create or edit.

1. **Slides are Quarto revealjs `.qmd`.** All examples and simulations use **R**.
2. **Tidyverse style**: `tibble`, `dplyr` verbs, `|>` or `%>%`, `tidyr` for reshaping, `purrr::map*()` or `replicate()` + tibble for simulation, `ggplot2` for all graphics. Avoid base-R plotting and `for` loops unless the loop itself is the teaching point.
3. **`echo: true` always** — never hide code. To save space use `#| code-fold: true`, not `echo: false`.
4. **Math in figures uses plotmath** (`expression()` / `bquote()`), never plain strings like `"theta_hat"`. Applies to titles, axis labels, legend labels, facet labels, and `annotate()`.
   - Static: `labs(x = expression(hat(theta)), y = expression(f(x*";"~theta)))`
   - Computed values: `annotate("text", x = 1, y = 2, label = as.expression(bquote(hat(p) == .(round(p_hat, 3)))))` — wrap `bquote()` in `as.expression()` so ggplot renders it reliably; a bare `bquote()` is fine inside `labs()`.
   - Legends: `scale_color_manual(values = ..., labels = c(expression(hat(theta)[1]), expression(hat(theta)[2])))`
5. **Examples are medical / biomedical** — clinical trials, biomarkers, lab values, adverse events, time-to-event, diagnostic tests.
6. **Also use textbook examples**, cited on the slide title: `## Example: Serum Transferrin Receptor (Devore 9.4, Exercise 55)` or `(Chihara 7.1.2)`. **Check every section and exercise number against the PDF** (paths under *Reference materials*) rather than recalling it.
7. **Each class is 2 hours** — roughly 40–60 content slides including R examples. Match the density of existing decks.
8. **Homework problems go at the end of each deck**, before References/Questions. Devore exercises are welcome, cited by section and exercise number.
9. **`set.seed(551)`** everywhere for reproducibility.

---

## Commands

```bash
quarto preview                        # live-reload local preview
quarto render                         # render everything into docs/
quarto render homework/HW_05.qmd      # single file — fastest edit loop
quarto render lessons/08_CI_One_Sample/08_CI_One_Sample.qmd
```

`Rscript class_dates_update.R` re-renders only the date-dependent pages (`homeworks.qmd`, `schedule.qmd`, every `homework/HW_*.qmd`). Run it after editing `class_dates.R` — those pages hardcode nothing, so a date change is invisible until they are re-rendered.

**Slide PDFs are made with decktape, not Quarto.** Each deck ends with a non-evaluated chunk holding its own export command; render the deck first, then run the command:

````
```{bash}
#| eval: false
#| include: false
decktape reveal --fragments docs/lessons/NN_Name/NN_Name.html lessons/NN_Name/NN_Name.pdf
```
````

---

## Architecture

**Dates flow from one file.** `class_dates.R` defines `first_day`/`last_day` from the OHSU academic calendar, builds `cal_dates` as a day-by-day sequence, then names every class meeting, homework deadline, quiz window, and exam date as an *index into that sequence* (`w4d1 = cal_dates[22]`, `hw5 = cal_dates[53]`, …). Every page showing a date does `source(here("class_dates.R"))` in its setup chunk and interpolates the variable, e.g. `` `r hw5 %>% format("%m/%d")` ``. To shift the term, change `first_day`/`last_day` and the indices — never edit dates in the `.qmd` files.

**Output is committed.** `output-dir: docs` in `_quarto.yml`, and `docs/` is tracked — GitHub Pages serves it from `main`. A content change is not live until the rendered HTML is committed too.

**Freeze is on.** `execute: freeze: auto` caches chunk results in the tracked `_freeze/` directory; pages re-execute only when their source changes. Commits touching only `_freeze/**/execute-results/*.json` are normal. If output looks stale despite a source edit, delete that page's `_freeze/` subdirectory and re-render.

**Freeze hides code that no longer runs.** A cached page is not evidence that its R still executes — the cache may predate a package upgrade. Editing one line of such a file invalidates its whole cache and re-executes every chunk, so an unrelated edit can surface errors that look like your fault but are not. Two real cases: a `label = bquote(...)` that newer ggplot2 rejects, and a PDF that had never once built. When touching a long-cached file, expect the first render to fail on something old, and check the error's line number before assuming it is your change. `Rscript -e 'knitr::purl(f); parse(...)'` is a cheap syntax pre-flight that avoids a slow render round-trip.

**The render list is the publish gate.** `_quarto.yml` carries an explicit `render:` list. Without it Quarto renders *every* `.qmd` in the project, which is how 30 superseded `_old`/`_long` drafts and leftover BSTA 550 material ended up on the live site. The list keeps those sources on disk but out of `docs/`. **Add new lessons and homework freely — they match `**/*.qmd` — but if you add a draft, name it `*_old*` or `*_long*` so it is excluded automatically.** Solution files *are* published and reachable by URL; they are gated only by not being linked from `homeworks.qmd` until released.

**Theming.** `sandstone_NW_JM.scss` is the site theme (the only theme in `_quarto.yml`); `sandstone_NW.scss` is the un-customized upstream copy. Slides use `lessons/simple_NW.scss`, referenced from decks as `../simple_NW.scss`. `styles.css` exists but is **not** wired into `_quarto.yml` — editing it has no effect unless you also add `css: styles.css` to the html format.

---

## File conventions

```
class_dates.R                      # all term dates
lessons/simple_NW.scss             # revealjs theme
lessons/NN_Topic/NN_Topic.qmd      # slide deck
lessons/NN_Topic/NN_Topic.pdf      # decktape export (committed, not built by quarto)
lessons/NN_Topic/NN_Topic_key_info.qmd   # optional per-lesson announcements
homework/HW_NN.qmd                 # assignment
homework/HW_NN_Solutions.qmd       # key
docs/                              # rendered site (committed)
```

- Lesson directories are numbered to match the `Lesson` column of `schedule.qmd`.
- `HW_*` and `Midterm_Exam*` declare both `html` and `pdf` formats; the `Final_Exam*` files are html-only.
- Files suffixed `_old`, `_old2`, `_long`, etc. are prior drafts kept in place — leave them alone.

**Hub pages are hand-maintained.** `schedule.qmd` is the master grid (week / date / lesson / slides / PDF links) and `homeworks.qmd` is the assignment + solutions table. Adding a lesson or homework means editing the source file *and* adding its row — nothing is generated from the filesystem. Both use the `iconify` shortcode (from the vendored `_extensions/mcanouil/iconify`) wrapped in `[...]{style="color:#f8f5f0;"}`. Copy an existing row rather than writing one from scratch.

### Lesson map

TB = Devore/Berk/Carlton section unless marked C&H. Date key = the `class_dates.R` variable in the file's `date:` field. Verified against the files and `schedule.qmd`; `schedule.qmd` wins if they drift.

| File | Topic | TB | date key |
|---|---|---|---|
| `00_Intro/00_Intro.qmd` | Welcome | — | `w1d1` |
| `01_Intro_Inference/01_Intro_Inference.qmd` | Intro to statistical inference; statistics | 6.1, 6.2 | `w1d1` |
| `02_Point_Estimation/02_Point_Estimation.qmd` | Point estimation; bias, variance, MSE | 7.1 | `w1d2` |
| `03_MVUE_Likelihood/03_MVUE.qmd` | Unbiased estimators, MVUE, likelihood theory | 7.1, 7.4 | `w1d2` |
| `04_MME_MLE/04_MME_MLE.qmd` | Method of moments and MLE | 7.2 | `w2d1` |
| `05_Sufficiency/05_Sufficient_Statistics.qmd` | Sufficient statistics | 7.3 | `w2d2` |
| `06_Review_Point_Estimation/…` | Review: point estimation | 7 | `w3d2` |
| `07_Confidence_Intervals/…` | Interval estimation (pivots, z-intervals) | 8.1 | `w3d2` |
| `08_CI_One_Sample/…` | One-sample t; population proportions | 8.1–8.4 | `w4d1` |
| `09_Bootstrap_Introduction/…` | Bootstrap introduction | 8.1–8.4 | `w4d2` |
| `10_Bootstrap_CI/…` | Bootstrap CIs | 8.5 | `w4d2` |
| `11_Review_Confidence_Intervals/…` | More CIs + review | 8 | `w5d1` |
| `12_Hypothesis_Testing_Introduction/…` | Hypotheses, test procedures, Type I/II errors | 9.1 | `w5d2` |
| `13_Tests_Mean_Proportion/…` | z and t tests for mean; tests for proportion | 9.2–9.3 | `w6d1` |
| `14_PValues_Power_SampleSize/…` | P-values; power and sample size | 9.4 | `w6d1` |
| `15_NeymanPearson_LRT/…` | Neyman–Pearson lemma; LRTs | 9.5 | `w7d2` |
| `16_FurtherHypothesisTesting/…` | Further aspects of testing; test–CI duality | 9.6 | `w8d2` |
| `17_BootstrapHypothesisTesting/…` | Bootstrap hypothesis testing | C&H 8 | `w8d2` |
| `18_TwoSampleTests/…` | Two-sample CIs and tests | 10.1–10.2, 10.4–10.5 | `w9d1` |
| `19_PairedData/…` | Analysis of paired data | 10.3 | `w9d2` |
| `20_BootstrapPermutation/…` | Bootstrap & permutation, two samples | 10.6, C&H 3, 5 | `w10d1` |
| `21_NonparametricInference/…` | Nonparametric inference | 14 | `w10d1` |
| `22_GaussMarkov_OLS/…` | Linear regression, Gauss–Markov, OLS | 12 | `w10d1` |
| `23_BayesianInference/…` | Intro to Bayesian inference | 15 | hard-coded `"2026-03-11"` |
| `24_CourseReview/…` | Course review (not on the posted schedule) | — | hard-coded `"2026-03-11"` |

`lessons/0x_Test_Distributions/03_Test_Distributions.qmd` is an extra deck not on the schedule.

---

## Slide template

### YAML header — copy exactly, updating lesson number, title, footer, and date key

```yaml
---
title: "Lesson NN: Topic Title"
subtitle: "BSTA 551: Statistical Inference"
author: "Jessica Minnier"
title-slide-attributes:
    data-background-color: "#006a4e"
date: "`r library(here); source(here('class_dates.R')); wXdY`"
format: 
  revealjs:
    theme: "../simple_NW.scss"
    chalkboard: true
    scrollable: true
    slide-number: true
    show-slide-number: all
    width: 1955
    height: 1100
    footer: Lesson NN - Short Topic
    html-math-method: mathjax
    highlight-style: atom-one
execute:
  echo: true
  warning: false
  message: false
---
```

### Setup chunk

```r
#| label: setup
#| include: false
library(tidyverse)
library(broom)
library(patchwork)
set.seed(551)
theme_set(theme_minimal(base_size = 18))
```

Occasional extras: `infer`, `MASS`, `DiagrammeR`, `here`. Prefer `broom::tidy()` for model and test output.

### Deck structure

1. `# Lesson NN: Title {background-color="#2c3e50"}`
2. `## Review: Where We Left Off` — bullets from the previous lesson, then `. . .` and numbered **Today's Goals**.
3. `## Roadmap for Today` — table with columns `| Topic | Key Idea | Textbook |`.
4. `::: callout-note` "Why This Matters" — medical research motivation.
5. Content sections, each opened by a colored section slide: `# P-Values (Devore 9.4) {background-color="#3498db"}`.
6. `# Putting It All Together {background-color="#16a085"}` — decision tree, summary table, complete worked medical example, simulation.
7. `# Key Takeaways and Looking Ahead {background-color="#2c3e50"}` — `::: callout-important` "Today's Main Points", common mistakes, `## Looking Ahead` naming the next lesson and its TB sections.
8. `# Homework Problems {background-color="#e67e22"}` — required going forward; older decks predate it.
9. `## References` inside `::: nonincremental` — Devore sections, Chihara sections, any primary medical papers.
10. `## Questions? {.center background-color="#3498db"}`, then "Thank you!"
11. The decktape export chunk (see *Commands*).

### Styling

- Section-slide colors in use: `#2c3e50` (lesson open and takeaways), `#3498db`, `#e74c3c`, `#8e44ad`, `#27ae60`, `#9b59b6`, `#16a085`, `#f39c12`, `#e67e22`.
- Formal statements go in callouts with a `##` title inside:
  - `callout-important` → definitions, theorems, key results
  - `callout-note` → motivation, context, "why this matters"
  - `callout-tip` → intuition, R tips
  - `callout-warning` → common mistakes, assumptions, caveats
- `. . .` for incremental reveals; `::: nonincremental` for lists that appear at once; `::: columns` / `::: {.column width="50%"}` for side-by-side.
- Proofs: state the theorem in a callout, then reveal steps with `. . .`.
- Figures: commonly `#| fig-width: 12`–`14`, `#| fig-height: 5`–`7`; combine panels with `patchwork`.
- Plot colors: `"steelblue"`, `"#e74c3c"`, `"darkgreen"`, `"orange"`; `scale_color_brewer(palette = "Set1")`.
- Simulations: build a tibble of replicates (`tibble(rep = 1:B) |> mutate(est = map_dbl(...))`), summarize with `dplyr`, visualize with `ggplot2`, and compare empirical to theoretical (bias, variance, MSE, coverage, size, power).

---

## Homework template

```yaml
---
title: "Homework N"
subtitle: "BSTA 551"
description: "Due: `r library(here); source(here('class_dates.R')); hwN %>% format('%m/%d')` at 11pm"
date-modified: "today"
format: 
  html:
    link-external-newwindow: true
    toc: true
  pdf: default 
editor_options: 
  chunk_output_type: console
---
```

Setup chunk: `knitr::opts_chunk$set(echo = TRUE)`, `library(here)`, `source(here("class_dates.R"))`.

Body:
- `## Directions` — "Same as previous instructions"; `set.seed(551)` for simulations; a link to the HW `.qmd` on GitHub; "*You must show all of your work to receive credit.*"; one sentence naming the lessons covered.
- `## Questions` → `### Problem K: Title (Devore Section X.Y, Exercise Z)`, a medical scenario with data inline as `$$X = \{...\}$$`, hints in italics, parts labeled `**(a)**`, `**(b)**`, … mixing derivation, R computation, and simulation. Give R scaffolds with `# Your code here`.
- Solutions: title "Homework N Solutions", setup loads tidyverse and sets `set.seed(551)`, each part restated in bold, then derivation (display math, bold step headers) and R code.

---

## Course facts

- **Meets** Mon & Wed, 10:00 AM–12:00 PM PT. No class Jan 19, Feb 11, Feb 16. In-person instruction ends Wed Mar 11; finals week Mar 16–18 has no in-person teaching; all coursework due **Thu Mar 19, 11 pm**.
- **Assessment:** 50% homework, 20% midterm (take-home), 20% final (take-home), 10% attendance (two exit tickets per week, complete at least 10). 17 classes total.
- **Homework grading is completion-based:** a check mark for turning in 75% of question parts *completed*, right or wrong. On-time submissions get feedback; late ones don't, but carry no penalty. Two no-questions-asked 3-day extensions per student.
- **Cadence:** usually assigned Wednesday, due the following Thursday 11 pm, submitted as PDF on Sakai.
- **AI policy:** students may use AI on homework to learn and may even ask for direct answers, but must not paste answers in verbatim and must not use it on exams.
- **Texts:** Devore, Berk & Carlton, *Modern Mathematical Statistics with Applications*, 3rd ed. (Springer 2021, doi:10.1007/978-3-030-55156-8) — focus Ch 6–10. Supplemental: Chihara & Hesterberg, *Mathematical Statistics with Resampling and R* (resampling topics); Casella & Berger *Statistical Inference* 2nd ed.; ModernDive; PennState STAT 415; U. South Carolina STAT 713 (Tebbs).

## Reference materials

Local PDFs, outside this repo:

- Devore/Berk/Carlton (primary, Ch 6–10):
  `/Users/minnier/Dropbox (Personal)/Work/OHSU/Teaching/BSTA 551v2/DevoreBerkCarlton_springer_modern_math_stats.pdf`
  (`.epub` alongside it in the same directory)
- Chihara & Hesterberg (bootstrap Ch 5/7, permutation Ch 3, testing Ch 8, estimation Ch 6):
  `/Users/minnier/Dropbox (Personal)/Work/OHSU/Teaching/BSTA 552/Chester/chesterismay-math432/Student Solutions/Mathematical Statistics with Re - Laura M. Chihara.pdf`

Check section and exercise numbers against these PDFs before citing them.

---

---

## Rolling over to a new term

Worked example: Winter 2026 → Winter 2027. Copy the folder, point `git remote set-url origin` at the new repo, then work through this list. Nothing here is automated — every item is a place a stale 2026 value hides.

**1. Dates — do this first, everything else depends on it.**
- `class_dates.R`: set `first_day` to the OHSU "Term begins" date and `last_day` to "Term ends".
- Re-derive every index. `cal_dates` is a day-by-day sequence, so `w4d1 = cal_dates[22]` only lands on the right weekday if the term starts on the same weekday. **Verify each `wXdY` prints a Monday or Wednesday** — holidays and a shifted start silently move everything.
- `year = "2026"` is defined but unused; update or delete it.
- Run `Rscript class_dates_update.R` to re-render the date-bearing pages.

**2. Hardcoded dates that bypass `class_dates.R`** — these will not update themselves:
- `lessons/23_BayesianInference/23_BayesianInference.qmd` and `lessons/24_CourseReview/24_CourseReview.qmd` — `date: "2026-03-11"`
- `lessons/00_Intro/00_Intro_key_info.qmd` — `date: "01-05-2026"`
- `homework/Final_Exam.qmd` (`2026-03-06`), `Final_Exam_Solutions.qmd` and `Final_Exam_Solutions_rubric.qmd` (`2026-03-20`), `Midterm_Exam.qmd` (`2026-02-02`)
- `syllabus.qmd` lines ~34 and ~166 — "March 11, 2026", "March 18, 2026", "March 19, 2026 at 11pm"
- `syllabus.qmd` → *Known Exceptions* — the holiday list (Jan 19, Feb 11, Feb 16) and the finals-week line
- `syllabus.qmd` → *Attendance Policy* — "There are 17 classes total"
- `index.qmd` — "Winter 2026", "Jan 5 - March 18", meeting days/time, and the office-hours block

**3. Term-specific links — all 19 Microsoft Forms URLs are single-use per term.**
- Exit-ticket forms, one per class meeting, in `schedule.qmd`
- The course-feedback form in `syllabus.qmd`
- The forms embedded in `lessons/00_Intro/00_Intro.qmd`
- The TA's Webex room in `index.qmd` (`ohsu.webex.com/meet/parkecha`) — changes if the TA changes
- Sakai assignment links, and the instructor/TA names and office hours in `index.qmd` and `instructors.qmd`

**4. Clear the build cache.** `_freeze/` is ~130 MB and still carries `lessons copy/`, `lessons_old/`, and BSTA 550 probability entries (`07_pmfs`, `05_Equally_likely_outcomes`, …). Delete the whole directory and let it rebuild; stale freeze output is the usual cause of a page that will not update.

**5. Decide what the drafts are for.** The `render:` list keeps 30 `_old`/`_long` files and the 550 leftovers unpublished but on disk. If they are not useful as reference, deleting them removes most of the repo's bulk (the repo is ~1.2 GB: 287 MB `docs`, 155 MB `lessons`, 130 MB `_freeze`).

**6. Housekeeping.** Three `.DS_Store` files are tracked (`git rm --cached` them and add to `.gitignore`); `homework/HW_07_Solutions.aux/.log/.tex` and five `.Rhistory` files are build litter; `_publish.yml` still names Nicky Wakim's quarto-pub site and can go; `index.qmd` reads "This is course introduces".

**7. Re-point the site.** Update the repo URL in `index.qmd`, the `jessicaminnier.com/BSTA551_W26/` iframes in `lessons/00_Intro/00_Intro.qmd`, and confirm GitHub Pages serves `docs/` from `main` in the new repo.

**8. Before publishing, check the render list.** Run `quarto inspect` and confirm the input list is only the lessons and homework you intend to publish. A new draft named without `_old`/`_long` in it *will* be published.

## Legacy content and known issues

- `hw_answers/`, `readings/`, `draft_schedule.qmd`, `quiz.qmd`, and `_course-settings.yml` are carried over from the BSTA 550 site. `draft_schedule.qmd` is a *probability* schedule; `hw_answers/` are 550 homework answers; `quiz.qmd` describes quizzes BSTA 551 does not give. All are excluded from the render list and no longer published — kept on disk only as reference.
- `homework/HW_08–HW_10.qmd`, `lessons/24_CourseReview/`, and `lessons/0x_Test_Distributions/` exist but were never assigned or scheduled; also excluded from the render list.
- `_publish.yml` still points at Nicky Wakim's quarto-pub ID and `nwakim.github.io/bsta-550-25F` — stale, and unused by the GitHub Pages `docs/` deploy.
- `quiz.qmd` references `q2_open` / `q2_close` and `HW_08`/`HW_09` reference `hw8`/`hw9`, none of which `class_dates.R` defines. All three are excluded from the render list, so this no longer breaks a build.
- **Lesson 18's deck cites "DBC 9.5" on several slides for two-sample z/t procedures.** That is wrong — Devore 9.5 is the Neyman–Pearson lemma; two-sample z/t is 10.1–10.2. `schedule.qmd` has been corrected; the in-deck citations have not.
- `_freeze/` still holds `lessons copy/`, `lessons_old/`, and BSTA 550 probability entries (`07_pmfs`, `05_Equally_likely_outcomes`, …). Harmless build cache, but ~130 MB of it.
