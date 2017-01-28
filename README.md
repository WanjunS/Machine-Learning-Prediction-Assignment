# Machine-Learning-Prediction-Assignment
1/27

## Background
#####Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Download Data
```
trainUrl <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
trainFile <- "pml-training.csv"
testFile  <- "pml-testing.csv"
if (!file.exists(trainFile)) {
  download.file(trainUrl, destfile=trainFile)
}
if (!file.exists(testFile)) {
  download.file(testUrl, destfile=testFile)
}
```

```
trainingDataSet <- trainingDataSet[,(colSums(is.na(trainingDataSet)) == 0)]
dim(trainingDataSet)
```
##### [1] 19622    60
```
testingDataSet <- testingDataSet[,(colSums(is.na(testingDataSet)) == 0)]
dim(testingDataSet)
```
##### [1] 20 60
##### Now I get a good understanding of how large the dataset is: training 19622, testing 20, variables 60 for both.


## Data Cleansing
```
isAnyMissing <- sapply(DTest, function (x) any(is.na(x) | x == ""))
isPredictor <- !isAnyMissing & grepl("belt|[^(fore)]arm|dumbbell|forearm", names(isAnyMissing))
predCandidates <- names(isAnyMissing)[isPredictor]
predCandidates
```
##### Run the dim again, data size remains unchange. 

## Classe
```
set.seed(1)
inTrain <- createDataPartition(y=data3$classe, p=0.60, list=FALSE)
training <- data3[inTrain,]
valid <- data3[-inTrain,]
```
##### Further partition the data into 60% training /40% validation.

## Train with random forest

## parallel clusters
```require(parallel)
require(doParallel)
cl <- makeCluster(detectCores() - 1)
registerDoParallel(cl)
```
## control parameters
```
ctrl <- trainControl(classProbs=TRUE,
                     savePredictions=TRUE,
                     allowParallel=TRUE)
```
## fit model and stop
```
method <- "rf"
system.time(trainingModel <- train(classe ~ ., data=DTrainCS, method=method))
stopCluster(cl)
```
```
Resampling: Cross-Validated (10 fold) 
Summary of sample sizes: 13245, 13248, 13245, 13245, 13245, 13247, ... 
Resampling results across tuning parameters:

mtry  Accuracy   Kappa    
2    0.9929331  0.9910609
14    0.9925931  0.9906307
27    0.9893992  0.9865911
```
##### mtry=2 gives the highest accuracy, chosen for final model.
```
trainingModel
hat <- predict(trainingModel, DTrainCS)
confusionMatrix(hat, DTrain[, classe])
hat <- predict(trainingModel, DProbeCS)
confusionMatrix(hat, DProbeCS[, classe])
save(trainingModel, file="trainingModel.RData")
```
##### confusion matrix shows the same conclusion.

## Cross Validation
```
predValidRF <- predict(modFitrf, validation)
confus <- confusionMatrix(validation$classe, predValidRF)
confus$table
```

```
out_of_sample_error <- 1 - modAccuracy
out_of_sample_error
```
##### [1] 0.003465658
##### out of sample error rate is very low across 5 validation classes.
```
accur <- postResample(validation$classe, predValidRF)
modAccuracy <- accur[[1]]
modAccuracy
```
##### [1] 0.9956634
##### The model achieved 99.6% accuracy.

## Predicting&Testing
```
pred_final <- predict(modFitrf, pre_testingDataSet)
pred_final
```

## Summary
Random forest gives the most ideal prediction results. The Confusion matrix achieved 99.6% accuracy. The re-sampling and cross function concept of random forest helps dealing with large data set.
