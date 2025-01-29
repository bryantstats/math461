---
title: "Math 461 - Final Project - Sample Codes"
format: 
  html: 
    toc: true
editor: visual
---




---

*This is a sample codes for part 4 of the Project only. For other parts of the Project, please refer to the assignments for sample codes.*

---

## 1. Binary Target

We will work with the [Titanic Dataset](Titanic.csv). The interested target variable is `Age`. Since this variable is numeric, we will create a binary variable from the `Age` variable and make it our target variable. 

- Logistic Regression


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
df = read_csv('Titanic.csv')

# Select variables for modeling
df = df %>%  
  select(Age, Survived, Pclass, Sex) %>% 
  drop_na()

# Create the binary target variable from Age
median_age = median(df$Age)
df$target = ifelse(df$Age > median_age, "0", "1")
df$Age = NULL  # drop the Age variable from the predictors


# convert all categorical variables to factor
df = df %>% 
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass),
         Sex = factor(Sex),
         target = factor(target))

# Train 
model = glm(target ~ ., 
            data = df,
            family = binomial(link = "logit"))

# Make predictions
predicted_prob = predict(model, df, 
        type = 'response')

pred = factor(ifelse(predicted_prob>=.5, 1, 0))

# calculate accuracy
cm <- confusionMatrix(data = pred, reference = factor(df$target))
cm$overall[1]
```

::: {.cell-output .cell-output-stdout}
```
 Accuracy 
0.6442577 
```
:::
:::



- Logistic Regression with PCA



::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)
df = read_csv('Titanic.csv')

# Select variables for modeling
df = df %>%  
  select(Age, Survived, Pclass, Sex) %>% 
  drop_na()

# Create the binary target variable from Age
median_age = median(df$Age)
df$target = ifelse(df$Age > median_age, "0", "1")
df$Age = NULL  # drop the Age variable from the data

# Change categorical variables to factor
df = df %>% 
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass),
         Sex = factor(Sex),
         target = factor(target))

# prepare the data for PCA
df_pca = df %>% select(-target)
df_pca = dummy_cols(df_pca, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

# PCA
res.pca <- prcomp(df_pca,  scale = TRUE)

# Create a dataset with only 2 PCs
df2 = as_tibble(res.pca$x[, c(1:2)])
df2$target = df$target
df = df2


# Logistic Regression
model = glm(target ~ ., 
            data = df,
            family = binomial(link = "logit"))

predicted_prob = predict(model, df, 
        type = 'response')

pred = factor(ifelse(predicted_prob>=.5, 1, 0))

cm <- confusionMatrix(data = pred, reference = df$target)
cm$overall[1]
```

::: {.cell-output .cell-output-stdout}
```
 Accuracy 
0.5966387 
```
:::
:::


- LDA


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)
df = read_csv('Titanic.csv')

# Select variables for modeling
df = df %>%  
  select(Age, Survived, Pclass, Sex) %>% 
  drop_na()

# Create the binary target variable from Age
median_age = median(df$Age)
df$target = ifelse(df$Age > median_age, "0", "1")
df$Age = NULL  # drop the Age variable from the data

# Change categorical variables to factor
df = df %>% 
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass),
         Sex = factor(Sex),
         target = factor(target))

# Encoding categorical to have a numeric dataset

df_predictors = df %>% dplyr::select(-target)
df_predictors = dummy_cols(df_predictors, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

df2 = cbind(df %>% dplyr::select(target), df_predictors)
df = as_tibble(df2)

model = MASS::lda(target ~ .,
            data = df)
# make predictions
pred = predict(model, df, 
        type = 'response')$class
# calculate accuracy
cm <- confusionMatrix(data = pred, reference = factor(df$target))
cm$overall[1]
```

::: {.cell-output .cell-output-stdout}
```
 Accuracy 
0.6442577 
```
:::
:::

- QDA


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)
df = read_csv('Titanic.csv')

# Select variables for modeling
df = df %>%  
  dplyr::select(Age, Survived, Pclass, Sex) %>% 
  drop_na()

# Create the binary target variable from Age
median_age = median(df$Age)
df$target = ifelse(df$Age > median_age, "0", "1")
df$Age = NULL  # drop the Age variable from the data

# Change categorical variables to factor
df = df %>% 
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass),
         Sex = factor(Sex),
         target = factor(target))

# Encoding categorical to have a numeric dataset

df_predictors = df %>% dplyr::select(-target)
df_predictors = dummy_cols(df_predictors, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

df2 = cbind(df %>% dplyr::select(target), df_predictors)
df = as_tibble(df2)

model = MASS::qda(target ~ .,
            data = df)
# make predictions
pred = predict(model, df, 
        type = 'response')$class
# calculate accuracy
cm <- confusionMatrix(data = pred, reference = factor(df$target))
cm$overall[1]
```

::: {.cell-output .cell-output-stdout}
```
 Accuracy 
0.6442577 
```
:::
:::



## 2. Numeric Target

In this section, our target will be the original `Age` variable, which is already numeric variable. 

- Linear Regression


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)

df = read_csv('Titanic.csv')

# prepare the data for modeling
df = df %>%  
  dplyr::select(Survived, Age, Pclass, Sex) %>% 
  rename(target = Age) %>%        # rename the target variable  
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass)) %>%  # fix types of variables 
  drop_na()                            # drop missing values

# Encoding categorical to have a numeric dataset

df_predictors = df %>% dplyr::select(-target)

df_predictors = dummy_cols(df_predictors, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

df2 = cbind(df %>% dplyr::select(target), df_predictors)
df = as_tibble(df2)

model <- lm(target ~ ., data = df)
summary(model)
```

::: {.cell-output .cell-output-stdout}
```

Call:
lm(formula = target ~ ., data = df)

Residuals:
    Min      1Q  Median      3Q     Max 
-39.631  -8.510  -0.756   8.154  46.944 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)   41.631      1.613  25.804  < 2e-16 ***
Survived_1    -6.331      1.257  -5.039 5.96e-07 ***
Pclass_2      -9.512      1.399  -6.797 2.27e-11 ***
Pclass_3     -15.966      1.269 -12.580  < 2e-16 ***
Sex_male       1.391      1.211   1.148    0.251    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 13.11 on 709 degrees of freedom
Multiple R-squared:  0.1906,	Adjusted R-squared:  0.186 
F-statistic: 41.74 on 4 and 709 DF,  p-value: < 2.2e-16
```
:::
:::


- PCA Regression


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)

df = read_csv('Titanic.csv')

# prepare the data for modeling
df = df %>%  
  dplyr::select(Survived, Age, Pclass, Sex) %>% 
  rename(target = Age) %>%        # rename the target variable  
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass)) %>%  # fix types of variables 
  drop_na()                            # drop missing values

# Encoding categorical to have a numeric dataset

df_predictors = df %>% dplyr::select(-target)

df_predictors = dummy_cols(df_predictors, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

df2 = cbind(df %>% dplyr::select(target), df_predictors)
df = as_tibble(df2)


# PCA
df_pca = df %>% dplyr::select(-target)
res.pca <- prcomp(df_pca,  scale = TRUE)

# Create a dataset with only 2 PCs
df2 = as_tibble(res.pca$x[, c(1:2)])
df2$target = df$target
df = df2

model = lm(target~., data = df)
summary(model)
```

::: {.cell-output .cell-output-stdout}
```

Call:
lm(formula = target ~ ., data = df)

Residuals:
    Min      1Q  Median      3Q     Max 
-33.478  -8.651  -1.759   8.133  50.034 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  29.6991     0.5331  55.707  < 2e-16 ***
PC1           0.7266     0.3878   1.874   0.0614 .  
PC2          -2.4797     0.4783  -5.184 2.83e-07 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 14.25 on 711 degrees of freedom
Multiple R-squared:  0.04098,	Adjusted R-squared:  0.03829 
F-statistic: 15.19 on 2 and 711 DF,  p-value: 3.46e-07
```
:::
:::


- Possion Regression


::: {.cell}

```{.r .cell-code}
library(tidyverse)
library(caret)
library(fastDummies)

df = read_csv('Titanic.csv')

# prepare the data for modeling
df = df %>%  
  dplyr::select(Survived, Age, Pclass, Sex) %>% 
  rename(target = Age) %>%        # rename the target variable  
  mutate(Survived = factor(Survived),
         Pclass = factor(Pclass)) %>%  # fix types of variables 
  drop_na()                            # drop missing values

# Encoding categorical to have a numeric dataset

df_predictors = df %>% dplyr::select(-target)

df_predictors = dummy_cols(df_predictors, remove_first_dummy = TRUE, remove_selected_columns = TRUE)

df2 = cbind(df %>% dplyr::select(target), df_predictors)
df = as_tibble(df2)

model = glm(target ~. , 
            data = df,
            family = poisson(link = "log"))


r2 = with(summary(model), 1 - deviance/null.deviance)

print(paste0("R-Squared: ", r2))
```

::: {.cell-output .cell-output-stdout}
```
[1] "R-Squared: 0.168958401321273"
```
:::
:::


---

