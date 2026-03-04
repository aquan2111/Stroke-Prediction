# Stroke-Prediction

## Summary
The dataset for this project comes from https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset/data.

In particular, the goal is to classify stroke given various factors, including age, gender, various diseases, and smoking status.

## Dataset information
The dataset contains 5110 observations with 12 attributes. The data is in good quality, with only one entry missing in the BMI column.

For the stroke column, which is the target feature column in this project, 4861 observations returned no stroke, while 249 observations returned stroke.

## Data preprocessing
The dataset is stratified split into 60-20-20, where 60% of the data is used for training, 20% for validation, and 20% for testing. Stratified split ensures that the percentage of stroke patients among each group is the same.

For categorical columns, one hot encoding was used for columns with more than 2 categories. After all columns become numerical/boolean, some techniques were used include interaction features and flagging extreme values (BMI or Average Glucose Level)

## Handling class imbalance
Different methods were used, including undersampling (Random Undersampling, TomekLinks, NearMiss), oversampling (Random Oversampling, SMOTE, ADASYN), and adjusting class weight inside each models

## Models experimented
Three types of models were used: Random Forest, XGBoost, and LightGBM. Each model is fitted on unsampled data and different sampled data.

Other ensemble methods were also tested, including stacking classifier and voting classifier.

Optuna is used for hyperparameter tuning.

## Evaluation
Due to class imbalance, accuracy is not used due to the results can be misleading.

The goal is to prioritize recall while maintaining a reasonable precision as in this projects, false negatives are more costly than false positives. In other words, minimizing incorrectly classified stroke patients is the key objective; however, incorrectly classified non-stroke patients are maintained at a reasonable level to minimize extra resources needed for these patients.

Therefore, instead of using F1 score, F2 score is the main metric used to evaluate these models as recall is more valued than precision, while still taking into account precision.

## Key results

While different sampling methods were used, eventually the best models are those fitted on unsampled data but with adjusted class weight. Different thresholds were also tested, but the threshold for these models with the best F2 score ends up being the default 0.5.

| Model | Attributes | F2 Score | Recall | Precision |
| ----- | ---------- | -------- | ------ | --------- |
| RandomForest | 'avg_glucose_level', 'age', 'bmi', 'age_ever_married', 'risk_count', 'age_hypertension', 'age_heart_disease', 'urban', 'male', 'work_type_private' | 0.4859 | 0.8158 | 0.1856 |
| XGBoost | 'age', 'smoking_status_unknown', 'ever_married', 'smoking_status_smokes', 'smoking_status_never_smoked', 'age_heart_disease', 'bmi', 'age_hypertension', 'work_type_private', 'age_ever_married' | 0.4704 | 0.7105 | 0.2 |
| LightGBM | 'avg_glucose_level', 'age', 'bmi', 'age_ever_married', 'urban', 'hypertension', 'male', 'work_type_self_employed', 'smoke', 'smoking_status_never_smoked' | 0.4909 | 0.7105 | 0.2195 |
| VotingClassifier | 'bmi', 'age', 'avg_glucose_level', 'age_ever_married', 'age_heart_disease', 'smoking_status_smokes', 'age_hypertension', 'work_type_private', 'ever_married', 'smoking_status_never_smoked', 'smoking_status_unknown', 'risk_count' | 0.5085 | 0.7895 | 0.2098 |

With the best model, it means that for approximately 1000 patients with 50 stroke cases, 39 are correctly flagged, while approximately 188 alerts are generated.

## Limitations

* Due to confidentiality, it is unknown whether the data is real or generated, and if the observations are real, where the data come from
* Deciding a threshold for key metrics, such as precision or recall, depends on actual operational constraints that each hospital may have, so the choice of F2 score in this scenario is only for reference, and may not be the actual metric used in real life scenarios
* Since precision is low at around 0.2, secondary models may be an option in further filtering of patients who were initially diagnosed with a stroke for better filtering