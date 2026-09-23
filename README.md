ATP Match Prediction
Predicting the winner of men's professional tennis matches using only pre-match information, comparing four statistical learning models in R.

Key result: All models reach about 65% accuracy and an AUC of ~0.71. Ranking-based features dominate the predictions; beyond that, match outcomes remain only moderately predictable from historical data.

Research Question:
How well can match results in men's tennis be predicted by player and match-related characteristics?
-> The task is framed as a binary classification problem: does player 1 win (y = 1) or player 2 (y = 0)?

Data:
- Source: ATP matches dataset (men's professional tennis, 1968–2022, ~188,000 matches, 49 variables) <!-- Add the link to the original data source here -->
- Scope: Matches from 2000–2020, restricted to hard courts to remove surface effects
- Final dataset: 25,612 matches with 55 features

Approach:
1. Preprocessing
Players randomly assigned to "player 1" / "player 2" (the raw data is stored as winner/loser), resulting in a balanced target variable
Player names and IDs removed so the models learn from player attributes, not identities
Post-match statistics and tournament-specific variables excluded to avoid leakage
Rows with missing values removed; data sorted chronologically
2. Feature Engineering
Normalized match statistics: counts converted to per-opportunity rates (e.g. ace %, serve win %) so long matches do not inflate values
Historical form without leakage: rolling averages over each player's previous 10 matches
Difference features: player 1 minus player 2 for each statistic
Ranking features: log_rank_diff (logarithmic rank difference, since the gap between rank 1 and 10 matters more than between 100 and 110) and rankpoints_diff
Handedness: one-hot encoded
Scaling: standardization fitted on the training set only
3. Models
Model	-> Tuning
Logistic Regression	-> Recursive Feature Elimination (best with 16 features)
Random Forest	-> Grid search over mtry, 5-fold CV, 500 trees
XGBoost -> Random search (30 combinations), 5-fold CV with early stopping
Neural Network	-> Grid search with 5-fold CV, 2 hidden layers, dropout, adaptive learning rate

Train/test split: 70/30.

Results: 
Performance of the tuned models on the test set:

Model	Accuracy	F1	AUC	Log Loss	Brier Score
XGBoost	0.624	0.699	0.713	0.620	0.216
Logistic Regression	0.649	0.651	0.712	0.620	0.216
Neural Network	0.646	0.631	0.709	0.622	0.217
Random Forest	0.650	0.657	0.705	0.746	0.277

Key findings: 
- All models perform on a similar level, suggesting a shared performance ceiling given the available pre-match data
- Tuning mainly helped XGBoost: the base model overfit heavily (train AUC ~0.95 vs. test ~0.71), the tuned model generalizes almost without a gap
- "log_rank_diff" and "rankpoints_diff" are by far the most important features; serve statistics contribute less, demographics (age, height) very little
- Factors like daily form, fatigue, injuries or mental state are not captured in the data, which limits predictability

Limitations & Possible Improvements
- A chronological train/test split (training on earlier seasons, testing on later ones) would reflect a real forecasting setting more closely than a random split
- Only hard-court matches were modeled; separate models per surface could be compared
- Additional features such as head-to-head records, fatigue (matches played in recent days) or Elo ratings could raise the ceiling

Project Structure: 
├── DataPreprocessing.R                  # Filtering, cleaning, player assignment
├── Statlearn_TennisClassification.R     # Feature engineering, models, evaluation
├── Step1/                               # <describe>
├── Step2/                               # <describe>
├── Step3/                               # <describe>
├── Models/                              # <describe>
├── Visuals/                             # Figures (ROC curves, feature importance, distributions)
└── Docu/                                # Project report


How to Run
1. Download the ATP matches data (see Data) and place it in the project root
2. Install the required R packages:
   install.packages(c(/* add packages, e.g. "tidyverse", "caret", "randomForest", "xgboost", "keras" */))
3. Run DataPreprocessing.R, then Statlearn_TennisClassification.R

  
