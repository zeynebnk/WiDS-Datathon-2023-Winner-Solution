
This repository contains my annotated winner solution code and slide presentation to the 2023 Stanford Women in Data Science Datathon as a high schooler. The competition involved extreme weather forecasting and can be accessed at https://www.kaggle.com/competitions/widsdatathon2023, with my winner interview and insights walkthrough avaliable at https://www.youtube.com/watch?v=eCui74vnLPo. 

_________________________________________________________________________________________________
**Solution Summary:**

Hello all!

I'm Zeyneb, a junior at Saratoga High School and the High School Winner of this year's WiDS Datathon! I am also a WiDS Ambassador from the Bay Area. I have experience working with data science and machine learning in projects and research, yet this was a relatively new area of application for me--I usually work with NLP and text. I joined a little later on in the competition, about February. It was really and amazing oppertunity to learn and I was able to apply a lot of new skills and learn new methods working on such and interesting problem to solve real-world problems. I had a lot of fun, and I would love to thank everyone at WiDS for making these events so that people like me can really grow and experience real data science! I got to explore a lot of new things working on this, and I wanted to share my approach and findings in the problem! Please don't hesitate to contact me with any questions or comments!

**Data Exploration:**

Right from the start, the data presented some very interesting characteristics. There is a looong time gap between train and test data, a significant differnece in the value distritbutions of the features, and outliers that seems to have comparably more/less temperature change over time. Of course, these can result in some interesting behaviors in the models, explored later below.

**Interesting/Unintuitive Findings (?):**

In initial experiments, I label encoded features with ~50 distinct values in the train data as categorical features when building the model, then selected top (30%-15%) features based on shapley values. I also performed hyperparameter tuning with 3-5 fold cross validation (split based on location i.e., 'loc_group', or random split). The final output performed relatively poorly (RMSE ~1.5). Strangely , specifying categorical features explicitly into the CatBoost and LightGBM models decreased performance, as did location based splitting. Despite the risks of overfitting and data leakage, random splits and less feature selection seemed to do better.
Furthermore, high number of boosting iterations (20K+) with small learning rate also did better
I expected overfitting and shakeup in the private leaderboard but I decided to explore and experiment with these.
Experiments that DIDN'T make the final approach:

Several forecasts for target variable are included in the data. I wanted to use these forecasts as proxy for target. (Strangely, it seems forecasts are constant for the first half of each month). I calculated average daily distance between forecasts and target value from training data (location-day_of_year), and used this as a feature for both train and test. I also added this value to test forecasts to estimate target.
I experimented with TabNet and AutoGluon as well for the models. I also worked with some augmentation methods like TabGAN and MixUp to generate synthetic data similar to the test set with pseudolabels. This helped however I have had inconsistent results over multiple runs, so I decided to leave them for replicability.
Climate Region "Expert" Models: I have also experimented with building a CatBoost model for each climate region with complete training data. I assigned higher sample weights for a that climate region and used that model to predict that specific climate region's test data. For example, for climate region = 'BSk', I assigned sample weights for training data where if the region is BSk then I assigned 1, 0.33 otherwise. I built a CatBoost model and then I used this model to predict only climate region BSk. This also helped improve RMSE.

**Final Approach:**

A key method I used is "iterative pseudolabeling" the test data to incoperate into the training, explained further.
I primarilly worked with CatBoost and LightGBM models, combining train and pseudo labeled test data, and ensemble the resulting models with previous steps' ensembled model. I iterate over 2 times and finally ensemble with climate region experts. While ensembling, the idea that I used is favoring the most recent model more, and use the ensembled (new and previous step's model) predictions only if the absolute difference between these predictions is below a certain threshold that is set based on experiment. If they differ a lot then use the new model. Final Approach Steps:
Build LightGBM and CatBoost models, LGB0 and CB0.
Build CatBoost Model, CB1, on train data with another set of hyperparameters (2K iterations)
Pseudolabel test data from CB1 predictions and build CatBoost model, CB_PL0, with the same hyperparameters from step 2 on train+test data.
Ensemble LGB0, CB0 and CB_PL0 models and use these predictions to pseudolabel test data and build a CatBoost model CB_PL1 with the same CatBoost hyperparameters as Steps 2 and 3, but increase iterations to 25K. Also build LightGBM model, LGB_PL1. Ensemble these two models, also ensemble with the previously ensembled models. i.e., New predictions are from ensembling LGB0, CB0, CB_PL0, LGB_PL1 and CB_PL1.
Use the latest predictions to pseudolabel tets data and build climate expert models. Ensemble climate experts predictions with the latest ensemble to get the final submission.
The models run in just over an hour.
