# Feature Overview

The model uses 22 engineered features. This page describes the ideas behind them; exact definitions, weightings and constants are not published.

## Speed and form

- **Recency-weighted speed figure average.** The core form measure, weighting recent races, well-backed runs and fair conditions more heavily.
- **Recent best.** The horse's best figure in its last few races.
- **Surface-specific form.** The same averages and bests, using only races on today's surface within a recent window.
- **Distance-specific form.** The same, using only races at today's distance group.
- **Form confidence.** How well supported the average is: the number of races behind it and how consistent they were.

## Pace

- **Early pace.** A recency-weighted average of the horse's early pace figures.
- **Mid-race pace.** The same at the second call.

## Class and competition

- **Class movement.** Whether the horse is stepping up or down from the level it has been racing at.
- **Figures relative to class.** How the horse's figures compare with the typical figure for the class it raced in.
- **Speed × class interaction.** Highlights fast horses dropping in class.
- **Strength of winners faced.** A recency-weighted average of the winning figures in the horse's recent races: how strong the company has been.

## Field quality ("key race")

- **Next-out performance of rivals.** For each past race, how the first three finishers ran in their next start, combined into a rolling average. A race full of horses who went on to run big is a stronger race than its figure alone suggests. Only next-out races run before the race being predicted are used, so the feature never looks ahead.

## Margin

- **Scaled finishing margin.** How close the horse finished to the winner, compressed so one heavy defeat doesn't dominate, and scaled by the strength of the race. A version adjusted to be more intuitive was tested twice and performed worse, so the original form was kept.

## Recency and trip

- **Days since last start.** Measured from the horse's true previous start, including races removed from its figure history.
- **Trouble in the previous race.** Flags parsed from the chart comment for being blocked, steadied, pressured on the pace or breaking poorly.

## Kept for review, not used by the model

Several values are computed and shown in the output to support manual handicapping, but are not model inputs:

- trainer form over the last 10 and 30 days, and whether today is a first start for a new trainer;
- the full chart comment from the horse's previous race;
- how wide the horse raced last time and how it finished;
- the morning line and how the model's ranking compares with it.
