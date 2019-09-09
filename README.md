# Literacy Rates in India - A Statistical Analysis of Overall Literacy Rates Across India's 36 States and Union Territories

Project Participants: Ramtin Talebi, Rajet Vatsa, Paul Stainier, Mirai Shah

### Introduction

                  *“Education is the most powerful weapon which you can use to change the world.” - Nelson Mandela*

Education is a cornerstone of empowerment; it gives people agency and enables those who are disenfranchised to break out of cycles of poverty and low expectations. Unfortunately, in many developing countries around the world, illiteracy remains a prevalent issue. For India in particular, approximately one in four people are considered illiterate. This project, part of a Kaggle competition and the final project for a course in linear models, used a mixed effects linear model to gain a better understanding on the status of education in India.

Based on a district-by-district analysis of the hundreds of potential predictors in the Education in India database (https://www.kaggle.com/rajanand/education-in-india), we identified sets of attributes that are most strongly associated with and predictive of overall literacy rates.

### Approach

The original dataset had 625 observations and 819 variables, many of which were merely subcategories of overall district variables. We thus only considered the aggregate variables, such as total private schools rather than the number of private primary schools; we did this, in part, to ensure that the number of predictors would be substantially less than the number of observations. We settled on 25 variables, the complete list of which can be seen on page 1 of the Appendix above.

Ultimately, after performing EDA and structuring some messy data, we constructed a mixed effects linear regression model with carefully selected predictors, and significant interaction terms for our final model, yielding a log-loss score of 0.58096.

### Exploratory Data Analysis

Preliminary visualizations of the quantitative variables show some noteworthy trends.

For instance, the age distribution of voters supporting Democrats vs. the age distribution of those not supporting Democrats in the 2016 election are quite different, with younger voters tending to support Democrats over older ones.

![Age Distribution vs. Democratic Support](md-images/Density-Plot-3.png)

In addition, voters living in more religious communities tended not to support Democrats while voters in less religious communities tended to support Democrats.

![Reiligous Distribution vs. Democratic Support](md-images/Density-Plot-2.png)

Lastly, it appears that individuals living in areas with lower median incomes were more likely to support Democrats.

![Income Distribution vs. Democratic Support](md-images/Density-Plot-1.png)

We also visualized the relationship between pairs of predictor variables to gain a sense of the best interaction terms. For instance, although we initially assumed a majority of veterans would support guns, we found that many veterans in fact had no interest in guns.

![Veterans vs. Gun Support](md-images/Collinearity-Plot-1.png)

### Dealing with Messy Data

Four predictors had missing values – age, percent of the county that is religious, percent of the county that is evangelical, and education status. While the first three variables only had a handful of missing values (19, 2, and 5 respectively), education had 943 missing values, equivalent to nearly 10% of the entire dataset.

We used two methods to deal with the missing data:

1) We originally handled this missing data using a na.convert.mean function. This method imputes missing values in quantitative predictors with the mean of the remaining data in that column, and adds an additional column of 1’s and 0’s that identifies whether or not an observation is missing a value for that specific predictor. The function handles categorical predictors by adding an extra level to the predictor that represents NA values.

2) Later, we regressed the available education data on the other predictors. We then used this regression to predict and impute values for the missing data using the “mice” package.

### Code Snippets — Model Validation

We first built a wide array of models, from basic GLMs (General Linear Models) with all 47 predictors to complex random forests with transformed variables and tuned parameters. After some initial validation, we settled on a generalized addiditive model (GAM) with smoothed predictors and interaction terms.

In order to train and validate that a GAM would yield the best predictions on the test set, we performed cross-validation, using the log-loss evaluation metric. This entailed fitting the model on a randomly chosen 90% of the training data, using the model to predict on the remaining 10% of the data, calculating the log-loss, and iterating 100 times, as follows:

```{r}
# Create looping
nsims=100
n.train=8000
n=nrow(train.na)
measure1=measure2=measure3=measure4=rep(NA,nsims)
for(i in 1:nsims){
  reorder=sample(n) 
  train=train.na[reorder[1:n.train],] 
  validation=train.na[reorder[(n.train+1):n],] 
  val_length = dim(validation)[1]
  
  # Four GAMs for validation
  model_final_gam = gam(suppdem ~ s(age) + sex + combined_ethnicity_4way + 
                          single + married + num_children + hasreligion + catholic + 
                          christian + interest_in_religion + donrever_1 + liberal_donor + 
                          conservative_donor + contbrel_1 + apparel_1 + boatownr_1 + 
                          cat_1 + environm_1 + outdgrdn_1 + guns_1 + s(cnty_pct_religious) + 
                          cnty_pct_evangelical + district_status + collapsed_educ + 
                          relative_income + liberal_donor*collapsed_educ +
                          combined_ethnicity_4way*catholic + sex*combined_ethnicity_4way +
                          guns_1*relative_income,
                        family = binomial, data = train, trace = FALSE)
  final_interaction_gam = gam(suppdem ~ s(age) + sex + combined_ethnicity_4way + 
                                single + married + num_children + hasreligion + catholic + 
                                christian + interest_in_religion + donrever_1 + liberal_donor + 
                                conservative_donor + contbrel_1 + apparel_1 + boatownr_1 + 
                                cat_1 + environm_1 + outdgrdn_1 + guns_1 + s(cnty_pct_religious) + 
                                cnty_pct_evangelical + district_status + collapsed_educ + 
                                relative_income + combined_ethnicity_4way*catholic +
                                num_children*relative_income + single*collapsed_educ +
                                single*collapsed_educ*s(age), 
                              family = binomial, data = train, trace = FALSE)
  model_final_gam_tuned = gam(suppdem ~ s(age) + sex + combined_ethnicity_4way + 
                                single + married + num_children + hasreligion + catholic + 
                                christian + interest_in_religion + donrever_1 + liberal_donor + 
                                conservative_donor + contbrel_1 + apparel_1 + boatownr_1 + 
                                cat_1 + environm_1 + outdgrdn_1 + guns_1 + s(cnty_pct_religious) + 
                                cnty_pct_evangelical + district_status + collapsed_educ + 
                                relative_income + liberal_donor*collapsed_educ +
                                combined_ethnicity_4way*catholic + sex*combined_ethnicity_4way +
                                guns_1*relative_income,epsilon = .001, bf.epsilon = .001,
                              family = binomial, data = train, trace = FALSE)
  final_interaction_gam_tuned = gam(suppdem ~ s(age) + sex + combined_ethnicity_4way + 
                                single + married + num_children + hasreligion + catholic + 
                                christian + interest_in_religion + donrever_1 + liberal_donor + 
                                conservative_donor + contbrel_1 + apparel_1 + boatownr_1 + 
                                cat_1 + environm_1 + outdgrdn_1 + guns_1 + s(cnty_pct_religious) + 
                                cnty_pct_evangelical + district_status + collapsed_educ + 
                                relative_income + combined_ethnicity_4way*catholic +
                                num_children*relative_income + single*collapsed_educ +
                                single*collapsed_educ*s(age), epsilon = .001, bf.epsilon = .001,
                              family = binomial, data = train, trace = FALSE)
  
  # Calculate predicted values for each
  predictions1 = predict(model_final_gam, newdata = validation, type = "response")
  predictions2 = predict(final_interaction_gam, newdata = validation, type = "response")
  predictions3 = predict(model_final_gam_tuned, newdata = validation, type = "response")
  predictions4 = predict(final_interaction_gam_tuned, newdata = validation, type = "response")
  
  validation$suppdem = ifelse(validation$suppdem=="Y", 1, 0)
  measure1[i]=-1/val_length * sum(validation$suppdem * log(predictions1) +
                                    (1 - validation$suppdem) * 
                                    log(1 - predictions1))
  measure2[i]=-1/val_length * sum(validation$suppdem * log(predictions2) +
                                    (1 - validation$suppdem) * 
                                    log(1 - predictions2))
  measure3[i]=-1/val_length * sum(validation$suppdem * log(predictions3) +
                                    (1 - validation$suppdem) * 
                                    log(1 - predictions3))
  measure4[i]=-1/val_length * sum(validation$suppdem * log(predictions4) +
                                    (1 - validation$suppdem) * 
                                    log(1 - predictions4))
}

# Calculate mean log-loss over all iterations
mean_model1 = mean(measure1)
mean_model2 = mean(measure2)
mean_model3 = mean(measure3)
mean_model4 = mean(measure4)
```
Our final log-loss from our chosen model, with smoothed predictors and interaction terms, was about 0.59.

### Model Diagnostics

To evaluate our model's susceptibility to outliers, we also did some diagnostic work, like evaluating Cook's distance:

![Cooks](md-images/Cook's-Distance.png)

Looks like our model holds up to outliers!

### Insights

Below are the effects of some of the predictors of our model:

![Coefficients](md-images/Coefficients.png)

Certain predictors had coefficient estimates that were not only interpretable, but consistent with established research. For example, liberal_donor has a large positive coefficient while guns_1 and conservative_donor had large negative ones. Beyond individual predictors that were either smoothed or left parameterized, some of the interaction terms are worth nothing as well. Research shows that the Catholic vote is generally split across party lines, but that minority Catholics generally are left-leaning. This might explain why ethnicity interacted with Catholic in our model has a large, positive coefficient.

### Conclusions and Improvements

In terms of our model's performance, it certainly could have performed better. We ran into problems of overfitting, which became clear when we dropped from 9th on the public leaderboard to 25th in the final rankings. By continuously revising our model until it was highly
ranked on the public leaderboard, we accidentally overfit our data to that small subset of the test data. To yield a better fitting model, one approach for future exploration is a more critical evaluation of the variables that appeared in the final model, since the greatest gains were realized following the reduction of the number of predictors in the dataset as well as the use of a revised missing data imputation approach.




Overall, our analyses showed that the mixed effects model (Model 4) was the best predictive model
of overall literacy rates. The summary table of Model 4 is shown in Figure 9, with 14 significant
continuous predictors and a random intercept based on state. A key finding from our regression
analyses, which was consistent with our original hypothesis, was the importance of factoring in the
state a district was located in. 
