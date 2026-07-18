# Marketing A/B Test: Conversion Analysis & ROI
 
Analysis of a real ad campaign A/B test — 588K users, split between seeing an actual ad vs a public service announcement (the control). Wanted a project that forced me to work with SQL directly and actually do proper experiment analysis, not just fit a model and report a metric. My other projects were all supervised learning; this one is closer to what a DA role actually asks you to do day to day.
 
## Data
 
Kaggle's Marketing A/B Testing dataset. 588,101 users, 564,577 in the ad group and 23,524 in the PSA (control) group — not balanced, but that's how the campaign was actually run, not something I could fix.
 
## SQL layer
 
Loaded the data into SQLite and did the aggregation in actual SQL, not pandas groupby, since I wanted this project to say something about SQL and not just Python: 
- Overall conversion rate by group
- Conversion rate broken out by hour of day (24 segments)
  
## What the hourly breakdown caught
 
A few early-morning hours (4am, 5am) had PSA sample sizes under 30 — at that size a single conversion swings the rate by several percentage points, so I flagged those hours as unreliable and excluded them before running anything that depended on hourly granularity. Everything else had at least ~80 PSA users per hour, mostly in the hundreds to low thousands during the day.
 
## Statistical test
 
Chi-square test on the full contingency table: p < 0.0001. At 588K users, though, almost any real difference clears that bar — so p-value alone doesn't tell you much here. Computed the effect size (Cohen's h) separately: 0.053, which is well under the 0.2 threshold usually called "small." The lift is real, but it's tiny.
 
Ran a power analysis to check whether the test was even well-powered enough to detect an effect this small: minimum sample needed was ~2,900 per group, actual PSA group was 23,524 — about 8x the minimum. So the significant result isn't an artifact of an underpowered test either.
 
## Turning it into a decision, not just a result
 
"Statistically significant" doesn't answer whether the campaign is worthrunning. Converted the conversion lift (2.56% vs 1.79%) into two break-evennumbers instead of picking arbitrary cost/revenue assumptions and reporting a single ROI figure:
 
- Break-even average order value**: $161.82 (at $0.05/impression)
- Break-even cost per impression**: $0.0154 (at $50 order value) Framed this way, the finding doesn't depend on numbers I made up — anyone with the real order value and media cost can immediately tell if this campaign was profitable. If your actual AOV is under $162 and your cost per impression is above $0.015, the ad spend outweighs the lift.
 
## What I'd do differently
 
Didn't have real cost/revenue data, so the break-even framing is the honest way to present this without pretending a made-up ROI number means anything. If I had actual business data, the next step would be running this break-even calculation against real cost curves instead of point estimates.
 
## Stack
 
Python, pandas, SQLite (sqlite3), scipy, statsmodels, matplotlib/seaborn
 
## TODO
 
- Segment the break-even analysis by hour, since conversion lift wasn't flat across the day (afternoon/evening hours had a noticeably bigger gap)
- Try a Bayesian approach to the A/B comparison instead of frequentist — would give a more direct probability statement about which group is better
- Look for other covariates in the raw data that might explain some of the hourly variation beyond just sample size noise
