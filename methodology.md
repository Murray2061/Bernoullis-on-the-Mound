# Methodology

A daily 2026 MLB pitcher ranking system built on a **Bernoulli reference model**, using **Suppression** probabilities to measure how rare a pitcher’s run-prevention line is. It also includes a structural uncertainty framework based on **combinatorial entropy** S = log2(Ω), which classifies performances into **Zen**, **Drama**, and **Meltdown** states.


## Overview  

Bernoullis on the Mound evaluates pitchers along two complementary axes.

The first is **suppression**, a probability-based score that asks how rare a pitcher’s run-prevention line is under a league-average Bernoulli reference model. The second is an **entropy-based decomposition** of the realized line into **Zen**, **Drama**, and **Meltdown**, designed to describe the **uncertainty** within the performance — that is, how outs and divided runs are structurally distributed and entangled within the line.

**Note on Project Evolution:** Earlier versions of this project (2025) explored **team doctrines** as a staff-level classification layer, but that first framework has been set aside. The project is developing a renewed **team doctrines** layer, intended as a higher-level interpretation built on top of suppression and entropy statistics, but that framework is still under construction.


## The Bernoulli Reference Model

At its core, this project treats every pitching performance  as a raw sequence of binary events: **Outs** and **Runs**.

Imagine a pitcher’s season as a long string of encoded results: `ooxoxxxoxoox...`. In this model, every **o** represents an out collected, and every **x** represents a divided run (DivR) allowed. This is a classic **Bernoulli trial**—a series of independent experiments where each event is a simple success or failure in the eyes of run suppression.

To judge these sequences, we aggregate the performance of every pitcher in the league to create a single, imaginary benchmark: the **MLB Bernoulli Pitcher**. This ideal pitcher represents the perfect mathematical average of the current season.

We then evaluate every real-world pitcher by asking: If we let this idealized MLB Bernoulli Pitcher take the mound for the same number of innings, what are the odds he would accidentally produce a line as good as—or better than—the one we just witnessed? This probability is what we call **Suppression**.


## Suppression

**Suppression** is the main ranking metric in this project. It measures how hard a pitcher’s line would be for the **MLB Bernoulli pitcher** to reproduce.

More precisely, suppression is the cumulative probability from the corresponding **negative binomial distribution**. The inputs are all measured from real data: the pitcher’s own outs and divided runs, and the league-level Bernoulli rate taken from the current MLB run environment. There are no free parameters.

**Lower suppression is better.** A very low suppression value means the line is rare under league-average conditions, so the pitcher is showing strong run suppression. A high suppression value means the line is easier for the league-level reference pitcher to reproduce, so the suppression is weak.

Because the baseline is built from the live relationship between runs and innings pitched across MLB, suppression always reflects the current scoring environment rather than a fixed historical era.


## Divided Runs (DivR)

This project uses **Divided Runs (DivR)** instead of assigning the full run to only one pitcher. When a runner is put on base by one pitcher and later allowed to score by another, the run is split **50/50** between the **On-Base Pitcher** and the **Scoring Pitcher**.

The goal is simple: to represent run responsibility more fairly across the full chain of events. Traditional run accounting is often too binary for this purpose. DivR keeps the model closer to the actual sequence of play, especially when multiple pitchers contribute to the same scoring outcome.

The automatic extra-inning runner follows the same logic. In those cases, the inherited runner is assigned to the **(GHOST)** pseudo-pitcher, which means **(GHOST)** can also receive partial divided runs such as **0.5 R** under the same 50/50 rule. This keeps the accounting consistent while preventing rule-created runners from being treated as if they were fully the responsibility of a human pitcher.


## Tier Landmarks

To make the raw decimal probabilities of the suppression scale human-readable, the model uses **Tier Landmarks** (S, A, B, C, and D).

The **S / A / B / C / D** tiers are fixed reference landmarks on the same suppression scale. Their purpose is to provide stable mental anchors, so a raw suppression value can be interpreted more intuitively and consistently. For example, the **S-Tier** benchmark is the probability of a league-average arm throwing a 9.0 IP shutout.

Each tier is anchored by a corresponding **dummy pitcher** line:

- **S-tier dummy:** 9.0 IP, 0 R
- **A-tier dummy:** 8.0 IP, 1 R
- **B-tier dummy:** 7.0 IP, 2 R
- **C-tier dummy:** 6.0 IP, 3 R

These dummy lines are included in the table as stable reference points on the suppression scale.

While the exact mathematical thresholds are updated to reflect the 2026 MLB scoring environment, these landmarks remain broadly similar to the 2025 version.


## Entropy states: Zen, Drama, and Meltdown

In addition to suppression, this project also describes each pitching line in terms of three entropy-based state components: **Zen**, **Drama**, and **Meltdown**.

The main purpose of this layer is to capture **uncertainty** in the structure of the line. In many statistical settings, uncertainty is described by **variance**, but variance is not a good fit here because it depends too strongly on how runs are recorded. For example, a two-run home run could be written as two separate one-run events or as a single two-run event. Those two notations would produce different variances, even though they describe the same baseball outcome.

So instead of measuring spread in run values, this model measures uncertainty in the arrangement of outs and runs within a fixed line. Remember that a pitcher’s line is treated as a sequence made from two event types: `o` for outs and `x` for divided runs.  The uncertainty considered here is about the structural complexity of how outs and divided runs are arranged within the line. Once the total numbers of outs and divided runs are fixed, the relevant question becomes: how many distinct sequences are compatible with that same performance?

That count is a combinatorial object, so the natural uncertainty measure is combinatorial entropy S = log2(Ω).

This entropy measures how strongly a pitcher’s runs and outs are entangled within the line. In baseball terms, it reflects the kind of performance that makes fans sweat, pace, and consider filing a complaint against their own bullpen-induced blood pressure. For that reason, this component is called **Drama**; its unit is **bits**.

The full line also has a **total bit count**: every out and every run contributes one event, so **total bits = outs + runs**. After the **Drama** part takes away the tangled, heart-attack portion of the line, the remaining bits are split by the pitcher’s **outs-to-runs ratio**. The out-heavy share is called **Zen** — the clean, under-control part of the performance. The run-heavy share is called **Meltdown** — the clear damage part, where the line stops being tense and just starts hurting.


## Reading the statistics

**Suppression** is the main ranking score. It measures how hard a pitcher’s line would be for the league-level **MLB Bernoulli pitcher** to reproduce, so **lower is better**. In the table above, **José Soriano** ranks first because his suppression is **0.00097189**, which is about **0.1%**; by comparison, the **Dummy-S** line (9.0 IP, 0 R) has a suppression of **0.01599666**, or about **1.6%**, so Soriano’s line is much rarer.

The three entropy states describe the internal shape of the line. **Zen** is the calm, out-dominant part. **Drama** is the tangled, stressful part, where outs and runs are mixed together in a way that makes fans sweat. **Meltdown** is the run-dominant part, where the line stops being tense and starts becoming obvious damage.

These numbers are meant to be read together. A pitcher above the **Dummy-S** line with very high **Zen** is not just suppressing runs well, but doing it cleanly. A pitcher with strong suppression but noticeably higher **Drama** may still be excellent, but in a more unstable and heart-attack-inducing way. For example, **Edward Cabrera** is a pure **100% Zen** line with no runs at all, while **Max Fried** is still outstanding by suppression, but with much more **Drama** and a small **Meltdown** share — meaning the result was strong, but the ride was bumpier.

One useful pattern is that S-tier and A-tier lines are usually strong sources of high-quality Zen.

Zen is tied to outs, which means it reflects game progress, and those top-tier lines usually accumulate that progress with relatively little Drama. By contrast, D-tier lines can still produce some Zen, but they often carry so much Drama that the value of that progress is heavily polluted by stress and instability. Meltdown, on the other hand, is usually the smallest component across baseball unless a pitcher is having a truly disastrous outing; one extreme example in the table is Carlos Estévez, whose line is so damaged that Meltdown rises above 50%.


## What This Model Is and Is Not

This model is a **descriptive ranking framework** for pitcher performance. It is designed to measure how rare a pitching line is under the current MLB run environment, and to describe the internal shape of that line through **Zen**, **Drama**, and **Meltdown**. One of its main goals is to provide a **universal benchmark** that allows different kinds of pitchers to be compared on the same scale, rather than forcing every comparison to stay trapped inside traditional role categories such as starters and relievers.

What it does **not** do is fully isolate pitcher skill in a causal sense. It does not try to separately remove the effects of defense, catcher framing, park conditions, opponent quality, sequencing context, or game situation. It also does not claim that the Bernoulli reference model is a literal physical law of baseball. It is a deliberately simplified baseline, built to produce interpretable rankings without hidden tuning parameters.

That simplicity is part of the point. The model is not trying to explain everything. It is trying to provide a clean, consistent language for comparing pitcher lines across the league — including lines that would usually be separated by role, usage, or scoring convention — and to do so with one common benchmark instead of five different statistical dialects.


## Data source

The rankings generated by this model are derived from **play-by-play data** sourced from [Baseball-Reference](https://www.baseball-reference.com/).
