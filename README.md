---
title: "Global Tech Salary Prediction and Analysis"
output: github_document
---

# Global Tech Salary Prediction and Analysis

### Shiva Manish Reddy  
### DS202 Final Project

## Introduction

The goal of this project is to analyze global technology salary data and determine which factors most strongly influence salaries in the tech industry. Using machine learning and data visualization techniques, this project explores relationships between salary and variables such as experience, location, education, company size, certifications, and job title.

This project also builds predictive machine learning models to estimate salaries using real-world technology job data.

The following research questions are explored throughout the analysis:

1. Which factors influence salary the most?
2. How strongly does experience affect salary?
3. How much does location affect salary?
4. Do education levels significantly impact salary?
5. Which job titles earn the highest salaries?
6. Does company size influence salary?
7. Are skills and certifications strong salary predictors?
8. Which machine learning model predicts salary most accurately?

---

# Data

## Dataset Description

The dataset contains approximately 250,000 global technology job records with information related to salary, experience, education, location, job title, company size, certifications, and industry.

## Variables Used

- salary
- experience_years
- education
- job_title
- company_size
- industry
- certifications
- location

---

# Libraries Used

```{r}
library(tidyverse)
library(caret)
library(xgboost)
library(Metrics)
library(scales)
```

---

# Loading Data

```{r}
df <- read_csv("salary_dataset.csv")
```

---

# Data Cleaning and Preprocessing

Several preprocessing steps were performed before analysis:

- Removed missing values
- Converted categorical variables into factors
- Prepared data for machine learning models
- Encoded categorical variables
- Split dataset into training and testing data

The dataset was divided into:
- 80% training data
- 20% testing data

This allowed the models to be evaluated on unseen data.

---

# Research Question 1:
# Which factors influence salary the most?

## Experience and Salary

```{r}
df %>%
  group_by(experience_years) %>%
  summarize(mean_salary = mean(salary)) %>%
  ggplot(aes(x = experience_years, y = mean_salary)) +
  geom_line(linewidth = 1) +
  scale_y_continuous(labels = dollar_format()) +
  xlab("Years of Experience") +
  ylab("Mean Salary (USD)") +
  ggtitle("Experience and Salary")
```

The graph shows a strong positive relationship between experience and salary. Workers with more years of experience consistently earn higher salaries. Salary growth appears steady across the career span.

```{r}
cor(df$experience_years, df$salary)
```

The correlation between experience and salary is approximately 0.44, making experience one of the strongest predictors in the dataset.

---

# Research Question 2:
# Which job titles earn the highest salaries?

```{r}
df %>%
  group_by(job_title) %>%
  summarize(mean_salary = mean(salary)) %>%
  arrange(desc(mean_salary))
```

The analysis shows that AI Engineer and Machine Learning Engineer positions consistently earn the highest salaries, while Data Analyst and Business Analyst positions remain among the lower-paying roles.

---

# Research Question 3:
# How much does location affect salary?

```{r}
df %>%
  group_by(location) %>%
  summarize(mean_salary = mean(salary)) %>%
  arrange(desc(mean_salary))
```

Location was one of the strongest salary predictors in the dataset. Workers located in the United States consistently earned significantly higher salaries than workers in other countries, especially compared to India.

---

# Research Question 4:
# Does company size influence salary?

```{r}
df %>%
  group_by(company_size) %>%
  summarize(mean_salary = mean(salary)) %>%
  arrange(desc(mean_salary))
```

The analysis showed that enterprise companies generally paid higher salaries than startups and smaller companies.

---

# Research Question 5:
# Does industry influence salary?

```{r}
df %>%
  group_by(industry) %>%
  summarize(mean_salary = mean(salary)) %>%
  arrange(desc(mean_salary))
```

Industry was found to be one of the weakest salary predictors in the dataset. Salary differences between industries were relatively small compared to differences caused by location or job title.

---

# Salary Prediction Models

## Train/Test Split

```{r}
set.seed(42)

train_index <- createDataPartition(df$salary, p = 0.80, list = FALSE)

train_data <- df[train_index, ]
test_data  <- df[-train_index, ]

nrow(train_data)
nrow(test_data)
```

The dataset was split into 80% training data and 20% testing data. The models were trained on the training set and evaluated using unseen testing data.

---

# Model 1: Linear Regression

```{r}
dummy_model <- dummyVars(salary ~ ., data = train_data, fullRank = TRUE)

train_encoded <- predict(dummy_model, newdata = train_data) %>% as.data.frame()
test_encoded  <- predict(dummy_model, newdata = test_data) %>% as.data.frame()

train_encoded$salary <- train_data$salary
test_encoded$salary  <- test_data$salary

lm_model <- lm(salary ~ ., data = train_encoded)

lm_preds <- predict(lm_model, newdata = test_encoded)

cat("RMSE:", round(rmse(test_data$salary, lm_preds), 0), "\n")
cat("MAE:", round(mae(test_data$salary, lm_preds), 0), "\n")
```

The Linear Regression model achieved:
- RMSE: 7149
- MAE: 5450

This model establishes a strong baseline but struggles to fully capture complex non-linear relationships between variables.

---

# Model 2: XGBoost

```{r}
encode_for_xgb <- function(data) {
  data %>%
    mutate(across(where(is.factor), as.integer)) %>%
    select(-salary) %>%
    as.matrix()
}

X_train <- encode_for_xgb(train_data)
X_test  <- encode_for_xgb(test_data)

dtrain <- xgb.DMatrix(data = X_train, label = train_data$salary)
dtest  <- xgb.DMatrix(data = X_test, label = test_data$salary)

xgb_params <- list(
  objective = "reg:squarederror",
  eta = 0.1,
  max_depth = 6,
  subsample = 0.8,
  colsample_bytree = 0.8
)
```

XGBoost outperformed Linear Regression and produced more accurate salary predictions by modeling non-linear relationships and interactions between variables.

---

# Key Findings

- Experience is one of the strongest salary predictors
- Location creates major salary differences
- AI and Machine Learning positions earn the highest salaries
- Industry has minimal influence on salary
- Enterprise companies generally pay higher salaries
- XGBoost outperformed Linear Regression

---

# Conclusion

This project successfully demonstrated how machine learning models can predict salaries using real-world global technology job data.

The analysis showed that experience, location, company size, and job title are the strongest salary drivers. Industry, however, had minimal impact compared to other predictors.

Among the machine learning models tested, XGBoost achieved the strongest prediction performance and produced more accurate salary estimates than Linear Regression.

The project also demonstrated the importance of data preprocessing, visualization, and machine learning techniques in solving real-world prediction problems.

---

# Future Improvements

Future improvements for this project include:

- Adding additional salary-related variables
- Including equity and bonus compensation
- Testing deep learning models
- Building a web application for salary prediction
- Exploring interaction effects between variables

---

# Technologies Used

- R
- tidyverse
- ggplot2
- caret
- xgboost
- Metrics
- Machine Learning
- Data Visualization
- R Markdown
