# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.
- 2026-11-21: Evaluated `cnt_change_1` using ride counts from two days prior to forecast. Doing so resulted in R^2 falling from 0.6475 to 0.6455 for Ridge, and from 0.2185 down to 0.2026 for HGB. While this is kept in the submitted experiment, it is reccomended to remove this from the deployed model.
- 2026-11-25: Added `/seed-fault` to `CLAUDE.md`. Updates the skill to append advice, leaving prior entries as is, and records hook output that was observed. 
- 2026-11-25: I checked HGB, and determined the target transformation to be a defensible decision, given that yesterday’s ride count is available at forecast time, allowing for analysis on temporal trends. Predicting the change based on the prior days count resulting in reduced validation MAE, dropping from 1308.9 to 833.4 rides. This transformation was kept as a separate experiment.