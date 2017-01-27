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
## Data Cleansing
```
isAnyMissing <- sapply(DTest, function (x) any(is.na(x) | x == ""))
isPredictor <- !isAnyMissing & grepl("belt|[^(fore)]arm|dumbbell|forearm", names(isAnyMissing))
predCandidates <- names(isAnyMissing)[isPredictor]
predCandidates
```
## Classe
```
set.seed(1)
inTrain <- createDataPartition(y=data3$classe, p=0.60, list=FALSE)
training <- data3[inTrain,]
valid <- data3[-inTrain,]
```
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
## Apply to both datasets
```
trainingModel
hat <- predict(trainingModel, DTrainCS)
confusionMatrix(hat, DTrain[, classe])
hat <- predict(trainingModel, DProbeCS)
confusionMatrix(hat, DProbeCS[, classe])
save(trainingModel, file="trainingModel.RData")
```

## Prediction
```
load(file="trainingModel.RData", verbose=TRUE)
DTestCS <- predict(preProc, DTest[, predCandidates, with=FALSE])
hat <- predict(trainingModel, DTestCS)
DTest <- cbind(hat , DTest)
subset(DTest, select=names(DTest)[grep("belt|[^(fore)]arm|dumbbell|forearm", names(DTest), invert=TRUE)])
```

## Summary
Random forest gives the most ideal prediction results. The Confusion matrix achieved 99.6% accuracy. The re-sampling and cross function concept of random forest helps dealing with large data set.
