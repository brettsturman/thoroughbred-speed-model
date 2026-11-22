# Methodology

This document explains the main design choices behind the model and the reasoning for each. Implementation details, exact formulas and tuned values are omitted.

## 1. The prediction task

The model forecasts the **speed figure** each horse will earn in today's race. A speed figure is a standardised measure of how fast a horse ran, adjusted for distance and track conditions, so it can be compared across races.

The target is a high percentile of each horse's likely figure (a "good day"), not its average. In a race, the winner is usually the horse that runs near the top of its range, so a ceiling-style forecast suits ranking better than a central one.

The rating is a forecast of performance, not a win probability. Ranking the field by it is the primary use.

## 2. Data

Past-performance data arrives as daily race cards. Each card lists every entrant and, for each one, a line per recent race: date, track, distance, surface, class, speed and pace figures, finishing margin, odds and the chart comment.

Appending cards over time builds a cumulative history. Because a horse's past races are repeated on every card it appears on, the first step is to collapse that history to **one row per horse per race date**, keeping the most recently received copy.

The card's own fields (today's track, class, distance and surface) describe the race the horse is entered in on that day, not the past race on the row. Keeping these two levels apart turned out to be one of the most important details in the pipeline.

## 3. Point-in-time training

Every historical race becomes one training example:

- **Target:** the figure the horse earned in that race.
- **Features:** only what was known beforehand: averages and bests from strictly earlier races, plus the race's own class, surface and distance.

This mirrors race day exactly, where features come from every race before today and the target is today's figure. Any feature that would look at the race itself, or later races, is excluded or shifted back.

## 4. Cleaning and exclusions

Some figures don't represent a horse's ability. Races where the horse was pulled up, eased, lost its rider, or was a heavily backed favourite beaten by a wide margin are removed from its **figure history**. Comment text is parsed to identify these cases.

Removed races still count as **starts**. A separate race calendar records every real start, so recency features such as days since last start reflect the horse's true schedule even when its last race was set aside.

## 5. Training targets: clean trips only

A race is excluded as a training **target** when its comment shows the horse's figure was compromised: a pace collapse, being checked or blocked, a bad break. The model therefore learns what a horse runs given a fair trip, rather than learning to predict bad luck.

## 6. Rolling averages

Most form features are recency-weighted averages over the horse's recent races. Each race's weight reflects:

- how recent it was;
- how well the horse was backed (runs at short prices count more);
- whether it was run on an off track;
- whether it came before a change of trainer.

Races with no value for the quantity being averaged carry no weight.

## 7. Model

- **LightGBM** gradient-boosted trees with a **quantile** objective.
- **Two models**, one for dirt and one for turf, each trained on races run on that surface.
- **Time-ordered cross-validation**: each fold trains on earlier dates and validates on later ones.
- **Recency-weighted training rows**, so recent racing counts more than older racing.
- Hyperparameters and rolling-average settings were tuned with Optuna. Validation scores proved insensitive to these settings across a wide range, so a fixed configuration is used day to day.

## 8. What the model deliberately does not use

- **Today's odds.** With odds as an input, the forecast largely echoes the market. Keeping them out produces an independent opinion; odds are shown beside the output for comparison.
- **Hand-set adjustments.** Post-prediction corrections for specific trainers or jockeys were removed. They were tuned on the same results used to judge them, rested on small samples, and did not update when the model was retrained.

## 9. Validation approach

- Development used time-ordered validation scores (pinball loss at the target quantile).
- Several scores reported during development were later found to be inflated by a data-leakage bug, which is a useful reminder that a good score is only meaningful once the pipeline is verified.
- Live evaluation now uses a **prediction log**: ratings are recorded before each race and never edited, and results are added afterwards. Measures include win and in-the-money rates by rank, performance by size of lead, and return at closing odds.

## 10. Known limitations

- Data covers only horses that have appeared on processed cards, so some rivals' histories are missing.
- International form depends on the data provider's conversion of foreign figures.
- First-time starters receive no rating.
