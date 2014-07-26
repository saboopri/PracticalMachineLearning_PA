Practical Machine Learning Project
========================================================
# Introduction

## Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: [LINK!](http://groupware.les.inf.puc-rio.br/har) (see the section on the Weight Lifting Exercise Dataset).

## Data
The training data for this project are available here: [Train Data](http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv)

The test data are available here: [Test Data](http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv)

## Goal
The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set.

# Report

## Data Processing
### Downloading and Reading Data 

```r
URLtrain<-"http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
URLtest<-"http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
filetrain<-"pml-training.csv"
filetest<-"pml_testing.csv"
download.file(URLtrain, filetrain)
download.file(URLtest, filetest)
train<-read.csv(filetrain, na.strings=c("NA", "#DIV/0!", ""))
test<-read.csv(filetest, na.strings=c("NA", "#DIV/0!", ""))
```

### Processing Data
#### Cleaning Data
*Removing NA's
*Rmoving first 7 variables as they cannot be used to predict the 'classe' variable

```r
nai<-apply(train, 2, function(x){sum(is.na(x))})
train<-train[, which(nai==0)]
train<-train[, -seq(1:7)]
nai<-apply(test, 2, function(x){sum(is.na(x))})
test<-test[, which(nai==0)]
test<-test[, -seq(1:7)]
```
#### Dividing Data for training and testing
Dividing training data into training(70%) and testing(30%) data to test the accuracy of the algorithm.

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.1.1
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
index<-createDataPartition(y=train$classe, p=0.7, list=FALSE)
traindata<-train[index,]
testdata<-train[-index,]
```
#### Processing Data

```r
preProc<-preProcess(traindata[-53], method="pca")
trainPP<-predict(preProc, traindata[,-53])
trainPP$classe<-traindata$classe
testPP<-predict(preProc, testdata[,-53])
testPP$classe<-testdata$classe
```


## Model Fitting
Using Random Forest Method as it is said to give the highest accuracy.

```r
model<-train(classe~., data=trainPP, method="rf", number=5, allowParalled=TRUE)
```

```
## Loading required package: randomForest
```

```
## Warning: package 'randomForest' was built under R version 3.1.1
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```
## Warning: package 'e1071' was built under R version 3.1.1
```
### Model

```r
model
```

```
## Random Forest 
## 
## 13737 samples
##    25 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## 
## Summary of sample sizes: 13737, 13737, 13737, 13737, 13737, 13737, ... 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy  Kappa  Accuracy SD  Kappa SD
##   2     1         1      0.003        0.004   
##   10    1         0.9    0.004        0.005   
##   20    0.9       0.9    0.007        0.008   
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 2.
```

## Cross Validation
### Predicted Values

```r
pv<-predict(model, testPP)
```
### Comparing the actual and Predicted Values

```r
comp<-pv==testPP$classe
summary(comp)
```

```
##    Mode   FALSE    TRUE    NA's 
## logical     125    5760       0
```
### Confusion Matrix

```r
confusionMatrix(pv, testPP$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1667   21    0    0    2
##          B    2 1104   13    3    4
##          C    2    8 1000   36    4
##          D    3    2   12  919    2
##          E    0    4    1    6 1070
## 
## Overall Statistics
##                                         
##                Accuracy : 0.979         
##                  95% CI : (0.975, 0.982)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : < 2e-16       
##                                         
##                   Kappa : 0.973         
##  Mcnemar's Test P-Value : 1.77e-05      
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.996    0.969    0.975    0.953    0.989
## Specificity             0.995    0.995    0.990    0.996    0.998
## Pos Pred Value          0.986    0.980    0.952    0.980    0.990
## Neg Pred Value          0.998    0.993    0.995    0.991    0.998
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.283    0.188    0.170    0.156    0.182
## Detection Prevalence    0.287    0.191    0.178    0.159    0.184
## Balanced Accuracy       0.995    0.982    0.982    0.975    0.993
```

**Accuracy**
the accuracy of algorithm is 97.84% as seen above in confusion matrix
**Sample Error**
the sample error for the algorithm is 2.16% (Sample Error = 100 - Accuracy)

## Applying Algorithm on Given Test Data

```r
pvtest<-predict(preProc, test[,-53])
answer<-predict(model, pvtest)
answer
```

```
##  [1] B A A A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
