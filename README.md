# PML_Project
This is the Github readme file, please refer to PML.html for the R mark down 
---


# Get the data 

load libraries  

```{r}
library(caret)
library(dplyr)
library(tidyr)
library(ggplot2)
```

Download the data from URL

```{r}
Url1 <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
Url2 <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

training <- download.file(Url1, destfile = "training.csv")
test <- download.file(Url2, destfile = "test.csv")
```
Read the data 
```{r}
training <- read.csv("training.csv")
test <- read.csv("test.csv")
```
# Clean the data
In order to run machine learning models, NA values are better to be eliminated, let's look at the data 
```{r}
colSums(is.na(training))
```
From the result, we see that some columns have no NA values, but others have a very high portion of NA values (19216/19622)
I think the best policy is not to throw away easily any data, but due to the high rate of NA values, these columns can't not be used for prediction 
Abandon the columns with NA values 

```{r}
training_cln <- training[colSums(is.na(training)) == 0]
test_cln <- test[colSums(is.na(test)) == 0]
```
The test data have more columns than the training data. 
In the training data we need "classe", in the test data we need "problem_id" 
We will further clean the dataset, by keeping the common columns
```{r}
features <- intersect(colnames(training_cln), colnames(test_cln))
training_cln <- training_cln[,c(features,"classe")]
test_cln <- test_cln[,c(features, "problem_id")]
```
it is essential to delete the first 7 rows, since they are either not numeric or irrelevant to prediction 
training_cln <- training_cln[,-c(1:7)]
test_cln <- test_cln[,-c(1:7)]
# Spliting the data
Now we are going to partition the dataset into train dataset and test dataset (60% - 40%), in order to calculate the sample error of the predictor. 
```{r}
set.seed(1233)

inTrain <- createDataPartition(training_cln$classe, p = 0.6, list = FALSE)
training_data <- training_cln[inTrain,]
testing_data <- training_cln[-inTrain,]
```
# Machine learning models
Next we are going to build three prediction models, using linear discriminant analysis, decision tree, random froest and generalized boosted model
we're going to use a 5 fold cross validation 
## LDA
Linear discriminant analysis 
```{r}
set.seed(1234)

control <- trainControl(method = "cv", 5)
fit_lda <- train(classe ~., data = training_data, trControl = control, method = "lda")
pd_lda <- predict(fit_lda, testing_data)
confusionMatrix(pd_lda, testing_data$classe)
conmx_lda <- confusionMatrix(pd_lda, testing_data$classe)$overall
```

## Decision tress
```{r}
set.seed(1235)
control <- trainControl(method = "cv", 5)
fit_dt <- train(classe ~., data = training_data, trControl = control, method = "rpart")
pd_dt <- predict(fit_dt, testing_data)
confusionMatrix(pd_dt, testing_data$classe)
conmx_dt <- confusionMatrix(pd_dt, testing_data$classe)$overall
```
## Gradient boosting model 
```{r}
set.seed(1236)
control <- trainControl(method = "cv", 5)
fit_gbm <- train(classe ~., data = training_data, trControl = control, method = "gbm")
pd_gbm <- predict(fit_gbm, testing_data)
confusionMatrix(pd_gbm, testing_data$classe)
conmx_gbm <- confusionMatrix(pd_gbm, testing_data$classe)$overall
```
#Random forest
```{r}
set.seed(1237)
control <- trainControl(method = "cv", 5)
fit_rf <- train(classe ~., data = training_data, trControl = control, method = "rf")
pd_rf <- predict(fit_rf, testing_data)
confusionMatrix(pd_rf, testing_data$classe)
conmx_rf <- confusionMatrix(pd_rf, testing_data$classe)$overall
```

# Conclusion 
Now let's compare the accuracy of these models 
```{r}
compare_accu <- data.frame(conmx_dt, conmx_gbm, conmx_lda, conmx_rf)
compare_accu
```
```{r}
                conmx_dt    conmx_gbm    conmx_lda  conmx_rf
Accuracy       0.5006373 9.578129e-01 7.054550e-01 0.9922253
Kappa          0.3469751 9.466096e-01 6.273989e-01 0.9901633
AccuracyLower  0.4895115 9.531280e-01 6.952299e-01 0.9900242
AccuracyUpper  0.5117626 9.621537e-01 7.155273e-01 0.9940479
AccuracyNull   0.2844762 2.844762e-01 2.844762e-01 0.2844762
AccuracyPValue 0.0000000 0.000000e+00 0.000000e+00 0.0000000
McnemarPValue        NaN 9.909286e-14 2.349934e-69       NaN
```
The best model if random forest 
```{r}
predict(fit_rf, test_cln)
 [1] B A B A A E D B A A B C B A E E A B B B
Levels: A B C D E
```
