# PSL EDGE v1 dataset

A 0–100 composite measure of franchise strength in the Pakistan Super
League, built from the same methodological family as MLC EDGE and IPL
EDGE — six independently interpretable components, season-adjusted,
frozen calibration for out-of-sample scoring.

Built from the uploaded Cricsheet JSON archive.

![PSL EDGE trajectories](visuals/psl_edge_trajectories.png)

## Coverage

12 seasons (2015/16–2026), 8 franchises, 72 team-season rows — but this is
an **unbalanced panel**, unlike MLC's static 6-team league:

- Multan Sultans joined in 2017/18.
- Hyderabad Kingsmen and Rawalpindiz are 2026-only entrants (shown as
  single diamond markers in the chart, not connected lines — one
  observation isn't a trajectory).

7 matches were excluded entirely from EDGE calculations for having no
recorded winner — this includes **both no-results and ties**, not just
rain-outs:

| Season | Match | Result |
|---|---|---|
| 2016/17 | Peshawar Zalmi vs Quetta Gladiators | no result |
| 2017/18 | Lahore Qalandars vs Islamabad United | tie |
| 2017/18 | Karachi Kings vs Lahore Qalandars | tie |
| 2019/20 | Karachi Kings vs Multan Sultans | no result |
| 2020/21 | Multan Sultans vs Karachi Kings | tie |
| 2021/22 | Peshawar Zalmi vs Lahore Qalandars | tie |
| 2025 | Lahore Qalandars vs Quetta Gladiators | no result |

Full list with match IDs: `excluded_matches.csv`.

## Method

- Unit: team-season.
- No-result matches are excluded entirely from EDGE calculations.
- Completed sample size `n` = count of team-match rows with a non-null
  winner/outcome.
- Ordinary metrics: shrink only when `n < 6` using
  `X* = n/(n+6)·X + 6/(n+6)·season_mean`.
- Fielding: always shrunk with `k=6` regardless of `n`, because
  named-fielder attribution is noisier than a sample-size problem alone.
- Pressure metrics: event-count shrinkage with `k=6`; teams with zero
  eligible events (no close games, no chases) revert fully to the season
  mean.
- All metrics z-scored within season.
- Lower-is-better metrics sign-inverted before averaging.
- Six components, equal-weighted (1/6 each) — not optimized against wins.
- Batting excludes redundant `run_rate` (`run_rate = 6 × runs_per_ball`).
- Run prevention uses `runs_allowed / legal_balls_bowled`.
- **EDGE 0–100** is an empirical-percentile transform fit only on
  **pre-2026** team-seasons in this league package; 2026 is not used for
  calibration — it's scored under the frozen pre-2026 transform, same
  out-of-sample discipline as MLC EDGE.

## Components

| # | Component | Metrics |
|---|---|---|
| 1 | Batting | runs_per_ball, dot_ball_rate_bat |
| 2 | Bowling | wickets_taken per match, dot_ball_rate_bowl |
| 3 | Run prevention | powerplay economy, death economy, runs allowed per legal ball |
| 4 | Explosiveness | boundary rate per ball, death run rate, powerplay run rate |
| 5 | Fielding | fielding dismissals per match |
| 6 | Pressure | close-game win pct, chase win pct, close-game run differential |

Note: PSL's pressure component includes **close-game run differential**
as a third metric, which MLC EDGE v1 dropped for simplicity (see MLC's
methodology doc). Keep this in mind if comparing component scores
directly across leagues — the pressure component isn't built identically
across the EDGE family yet.

## Files

- `team_match_edge.csv` — audit-level team-match inputs and derived rates.
- `team_season_edge_v1.csv` — final component scores and EDGE v1, all raw/shrunk/z-scored intermediates.
- `excluded_matches.csv` — matches excluded because no winner/outcome was recorded.
- `visuals/psl_edge_trajectories.png` — franchise EDGE by season.

## Results at a glance

Every top-2 finisher, every season 2015/16–2025:

| Season | 1st | 2nd |
|---|---|---|
| 2015/16 | Quetta Gladiators (94.5) | Peshawar Zalmi (64.8) |
| 2016/17 | Peshawar Zalmi (83.6) | Lahore Qalandars (57.0) |
| 2017/18 | Quetta Gladiators (85.2) | Islamabad United (74.2) |
| 2018/19 | Quetta Gladiators (86.7) | Islamabad United (82.0) |
| 2019/20 | Multan Sultans (97.7) | Islamabad United (58.6) |
| 2020/21 | Peshawar Zalmi (78.9) | Lahore Qalandars (75.8) |
| 2021 | Multan Sultans (96.1) | Islamabad United (68.0) |
| 2021/22 | Lahore Qalandars (93.0) | Multan Sultans (88.3) |
| 2022/23 | Multan Sultans (89.8) | Lahore Qalandars (77.3) |
| 2023/24 | Islamabad United (91.4) | Multan Sultans (69.5) |
| 2025 | Quetta Gladiators (99.2) | Lahore Qalandars (80.5) |
| **2026 (frozen, OOS)** | Peshawar Zalmi (100.0) | Hyderabad Kingsmen (79.7) |

Unlike MLC, no single PSL franchise dominates across the whole window —
the top finisher changes almost every season, and several teams
(Islamabad United, Multan Sultans, Peshawar Zalmi, Quetta Gladiators) have
each taken 1st at least twice. That volatility is itself a useful contrast
with MLC's much stickier Washington Freedom pattern once we get to
cross-league validation — a 12-season league gives EDGE far more chances
to be tested against real turnover than a 4-season one does.

## What this package does not do

Same non-goals as MLC EDGE v1: no recency weighting, no opponent
adjustment or in-season rolling updates, no weights fit to observed wins,
no roster/player-availability effects.
