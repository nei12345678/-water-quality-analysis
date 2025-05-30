
---
title: "Barbell Lift Prediction Project"
author: "Your Name"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(caret)
library(randomForest)
library(rpart)
library(ggplot2)
library(dplyr)
library(corrplot)
```

## Introduction

This project aims to predict the manner in which barbell lifts were performed using data from wearable sensors. The target variable `classe` represents the type of lift performed.

## Load and Clean Data

```{r load-data}
train_url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
test_url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

training <- read.csv(url(train_url), na.strings=c("NA","#DIV/0!",""))
testing <- read.csv(url(test_url), na.strings=c("NA","#DIV/0!",""))
```

### Remove NAs and Irrelevant Columns

```{r clean-data}
# Remove columns with too many NA values
training <- training[, colMeans(is.na(training)) < 0.95]

# Remove irrelevant columns (e.g., IDs, timestamps)
training <- training[, -(1:7)]
testing <- testing[, names(training)[-ncol(training)]]
```

## Data Partitioning

```{r partition-data}
set.seed(123)
inTrain <- createDataPartition(training$classe, p = 0.7, list = FALSE)
trainSet <- training[inTrain, ]
valSet <- training[-inTrain, ]
```

## Train Model (Random Forest)

```{r train-model}
fitControl <- trainControl(method = "cv", number = 5)
modelRF <- train(classe ~ ., data = trainSet, method = "rf", trControl = fitControl)
```

## Evaluate Model

```{r evaluate-model}
predRF <- predict(modelRF, newdata = valSet)
confusionMatrix(predRF, valSet$classe)
```

## Final Prediction on Test Set

```{r final-predictions}
finalPred <- predict(modelRF, newdata = testing)
finalPred
```

## Variable Importance

```{r importance-plot}
varImpPlot(modelRF$finalModel, main="Variable Importance (Random Forest)")
```

## Conclusion

The Random Forest model yielded high accuracy on validation data. Further improvements could include PCA, ensemble models, or deeper feature engineering.
