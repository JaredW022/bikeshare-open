## Reports

###   Results

| Feature set | Model | Validation R² | Validation MAE | Test R² | Test MAE |
|---|---|---:|---:|---:|---:|
| Without `cnt_change_1` | Mean baseline | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Without `cnt_change_1` | Persistence | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Without `cnt_change_1` | Ridge | 0.6475 | 832.9 | 0.5833 | 947.1 |
| Without `cnt_change_1` | HGB | 0.2185 | 1296.6 | -0.4975 | 1972.6 |
| With `cnt_change_1` | Mean baseline | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| With `cnt_change_1` | Persistence | 0.4507 | 900.6 | 0.5099 | 840.1 |
| With `cnt_change_1` | Ridge | 0.6455 | 833.4 | 0.5829 | 945.9 |
| With `cnt_change_1` | HGB | 0.2026 | 1308.9 | -0.5612 | 2021.2 |

### Headline finding

All models (with and without `cnt_change_1`), in both Test $R^2$, as well as in Test MAE. Notably, there were two models that seemed to performed well, being Ridge and Persistence. Persistance achieved the lowest MAE in both "Without `cnt_change_1`" and "With `cnt_change_1`" feature sets, achieving a Test MAE of 840.1 in both instances. The closest model was Ridge, with 947.1 and 945.9 respectively for "Without `cnt_change_1`" and "With `cnt_change_1`". This, however, was traded off with differences in Test $R^2$, with Ridge achieving the highest Test $R^2$ in both instances, at 0.5833 and 0.5829 respectively. `cnt_change_1` seemed to produce worse metrics, such as lower Test $R^2$ in Ridge, lower Validation $R^2$, higher Validation MAE, and only lower Test MAE. As such, I would likely exclude this from our final model, leaving just the documented experinment. 

### Exercise 4: target transformation

Gradient boosting was used to predict what the change between the ride count yesterday, adn todays ride count was. This was then added to the prior days ride count to predict what the current days ride count would be. From this, $R^2$ increased from 0.2026 to 0.6158 on the validation set, and MAE fell, from 1308.9 to 833.4. Given that we have access to the prior days information, this seems like a reasonable step, which accounts for possible temporal trends in the data.
After choosing model transformation, I evaluated it on the test set. Test $R^2$ increased, from −0.5612 to 0.5567, and Test MAE fell, with the original MAE being 2021.2 down to 922.1 rides. Persistance maintained the lowest MAE, at 840.1 rides.

### Exercise 5: ridge vs persistence

I would choose to deploy Persistance as our final model. I would choose so because it achieves lower MAE, meaning that the model correctly/ more accurately predicts the ride count value. This is demonstrated, as the average absolute error was 105.8 rides lower, relative to Ridge, per day, which came out lower than Ridge’s. MAE was chosen as the evaluation metric because it places higher importance on whether or not the prediction better approximated the true ride count.

### Attribution 
I used qwen3.5:9b through Ollama in Claude Code with a 64K context. I used this model thoughout the entire workflow process.

What the agents contributed
- Claude Code wrote the cnt_change_1 calculation in src/features.py, alonside the test for its expected value in tests/test_leakage.py. It also introduced the seeded fault, which after verification, was repaired. Tests were added for unknown weather labels and unclassified raw columns.
- Claude Code’s leakage-auditor generated the audit findings for my teammate’s repository. I read the findings and then drafted the final verdict.
- Codex checked my results table against the pipeline output.

Decisions I made
- I chose cnt_change_1, which represented whether ride counts was rising or falling prior to our prediction. This waas used in our calculation for Ridge and HGB, as an input column. This is primarily addressing temporal trends in the data, where our prediction may have been affected by external factors at a certain period of time. 
- I directed the HGB validation experiment. The main decision made was deciding whether this model was reasonable to implement, and given that we had access to prior day data, it seemed reasonable to analyze for possible temporal trends. This could have accounted for external factors which occured during a specific section of time, which result in differing predictions. 
- I chose which model would be deployed, reasoning between the two evaluation metrics, and ultimately deciding on valuing MAE over $R^2$.