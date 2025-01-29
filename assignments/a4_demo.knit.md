---
title: "Assignment 4: Logistic Regression in R"
format: 
  html: 
    toc: true
editor: visual
---




------------------------------------------------------------------------

**Submission**: Submit the knitted word document to Canvas.  

------------------------------------------------------------------------

## 1. Creating Logistic Regression

We will implement logistic regression on the [Hospital Cost Data](https://bryantstats.github.io/math461/assignments/HospitalCosts.csv). We want the total charge (`TOTCHG`) to be the response variable. Since the total charge is not a binary, we will tunr this variable to binary so that we can run a logistic repression on it.  

We will create a binary variable as called `charge`.  The `charge` takes values 0 if the patient's total charge is less then the median total charge and 1 otherwise. 

The codes are as follows. 


::: {.cell}

```{.r .cell-code}
d = read.csv('https://bryantstats.github.io/math461/assignments/HospitalCosts.csv')

# Calculate median charge
median_cost = median(d$TOTCHG)

# Create a column to compare the cost with median charge
# this column takes value 0 of the charge is less than the median and 1 otherwise

d$charge = ifelse(d$TOTCHG < median_cost, 0, 1)
```
:::


-   Now that the variable`charge` is binary, we can train a logistic regression with `charge` being a response.


::: {.cell}

```{.r .cell-code}
# Train
model = glm(charge ~ AGE + LOS, 
            data = d,
            family = binomial(link = "logit"))
# Report model summary
summary(model)
```

::: {.cell-output .cell-output-stdout}

```

Call:
glm(formula = charge ~ AGE + LOS, family = binomial(link = "logit"), 
    data = d)

Coefficients:
            Estimate Std. Error z value Pr(>|z|)    
(Intercept) -4.32670    0.44248  -9.778  < 2e-16 ***
AGE          0.06929    0.01858   3.729 0.000192 ***
LOS          1.76979    0.18351   9.644  < 2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 693.15  on 499  degrees of freedom
Residual deviance: 474.28  on 497  degrees of freedom
AIC: 480.28

Number of Fisher Scoring iterations: 7
```


:::
:::


## 2. Making Prediction

- Create a predicted charge column to store the predictions of the the model for each patients. 

-   Making prediction is similar to `lm`. For example, let's predict the probability of the cost larger than the median cost for a 20 year old who stayed 1 day at the hospital. 


::: {.cell}

```{.r .cell-code}
predicted_prob = predict(model, list(AGE = 20, LOS = 1), 
        type = 'response')
predicted_prob
```

::: {.cell-output .cell-output-stdout}

```
        1 
0.2366538 
```


:::
:::


-   To make a categorical prediction (predict 0 or 1), we can convert the probabilities to the category. If the predicted probability is less than 0.5, we classify it as 0 and 1 otherwise. The code goes as follows.


::: {.cell}

```{.r .cell-code}
predicted_class = ifelse(predicted_prob>=.5, 1, 0)
predicted_class
```

::: {.cell-output .cell-output-stdout}

```
1 
0 
```


:::
:::


## 3. Accuracy

How good is this model? One way to evaluate the model is to compare the predicted values and the true values of the charge (binary) and count the percentage of the correct prediction. This number is call the training accuracy of the model. 

We do it as follows.


::: {.cell}

```{.r .cell-code}
# Create 
d$predicted_prob_charge = predict(model, d, type = "response")

d$predicted_charge = ifelse(d$predicted_prob_charge <.5, 0, 1)

mean(d$predicted_charge == d$charge)
```

::: {.cell-output .cell-output-stdout}

```
[1] 0.814
```


:::
:::


## 4. Questions

a. Run the given codes and show its results.

b. Create a `LOS2` variable taking value 0 of the length of stay of the patient is fewer than the median length of stay and 1 otherwise. 

c. Create a logistic regression with response `LOS2`.

d. Calculate the training accuracy of the model. 

