---
title: "An AI Model Towards a Smarter Fitness"
author: "Pablo Sainz"
date: "2019-08-01"
output:
  html_document:
    keep_md: yes
---



## Abstract

Having a healthy lifestyle is not only about how much time is spent in fitness-related activities but also how well or efective is spent. What this paper will propose and address is to build a model based on Machine Learning techniques to be able to effectively classify and predict physical activity by a given subject and being able to determine which type of movement is being done.

## Target Data



The data sets on which the present work is based consist in the following:

- One training set of 160 variables with 19622 observations.
- One test set of 160 variables with 20 observations.
- Both data sets contains observations mapped to 5 classes.

## Data Preparation

While the initial presentation of the training set might look considerable, after an initial preprocessing it can be found that only 406 observations are complete. By completeness it is meant having values present on all the variables available.

Considering the above mentioned, then we proceed to filter out all the variables that contain empty or NA values and non useful columns for classification purposes as illustrated in the following code:


```r
# First we need to force the numeric format in the data frames to obtain the NA
numericTrainingSet <-rawTrainingSet[,sapply(rawTrainingSet, is.numeric)]
numericTestSet <-rawTestSet[,sapply(rawTestSet, is.numeric)]

# Once that we have all the non-existing values properly set to NA then we proceed to remove the 
# variables with NA values present
nonNATrainingSet <- numericTrainingSet[,colSums(is.na(numericTrainingSet)) == 0]
nonNATestSet <- numericTestSet[,colSums(is.na(numericTestSet)) == 0]
#Then add the classe column to the training set
nonNATrainingSet$classe = classeColumn

nonUsefulColsRegEx <- "X|timestamp"
usefulTrainingColumns <- !grepl(nonUsefulColsRegEx, colnames(nonNATrainingSet))
usefulTrainingColNames <- colnames(nonNATrainingSet)[usefulTrainingColumns]
targetTrainingSet = nonNATrainingSet %>% 
  select(usefulTrainingColNames)

usefulTestColumns <- !grepl(nonUsefulColsRegEx, colnames(nonNATestSet))
usefulTestColNames <- colnames(nonNATestSet)[usefulTestColumns]
targetTestSet = nonNATestSet %>% select(usefulTestColNames)
```

After that filtering now we have the following sets:

- One training set of 54 variables with 19622 observations.
- One test set of 54 variables with 20 observations.

Once that our initial filtering is prepared, then we proceed to create our validation data set. For the purposes of the present work, we will proceed with a split criterion of 80%-20% to enable our model validation, as illustrated in the following code:

```r
#Set a seed to enable a reproducible data split
inTrainPivot <- createDataPartition(targetTrainingSet$classe, p=0.80, list=F)
finalTrainingSet <- targetTrainingSet[inTrainPivot, ]
validationSet <- targetTrainingSet[-inTrainPivot, ]
```

## Proposed Model: Random Forest

Considering the present problem at hand, which is a classification problem, it has been proposed a classification model based on random forest techniques in order to provide a prediction model based on the provided data.

The proposed model is illustrated with the following code:

```r
# First establish a training control to make the training part less resource consuming
trainingControl <- trainControl(method = "cv",7,allowParallel = TRUE)
#Create the model
rfModel <- train(classe ~ ., data = finalTrainingSet,method = "rf", trControl=trainingControl,ntree=200)
```
## Model Results

Based on the model proposed on the previous section, we obtain a top accuracy of 0.9976431 per the comparison illustrated by he following figure:
![Model Training performance](Course8Project1_files/figure-html/unnamed-chunk-5-1.png)

Based on that, we proceed to predict on the created validation set using the generated model, obtaining the following performance as illustrated by the following confusion matrix:


```r
# Now get the predictor based on the trained model
rfPredictor <- predict(rfModel,validationSet)
#Then create the corresponding confussion matrix
confusionMatrix(validationSet$classe,rfPredictor)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1115    1    0    0    0
##          B    2  757    0    0    0
##          C    0    0  684    0    0
##          D    0    0    1  642    0
##          E    0    0    0    1  720
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9987         
##                  95% CI : (0.997, 0.9996)
##     No Information Rate : 0.2847         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.9984         
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9982   0.9987   0.9985   0.9984   1.0000
## Specificity            0.9996   0.9994   1.0000   0.9997   0.9997
## Pos Pred Value         0.9991   0.9974   1.0000   0.9984   0.9986
## Neg Pred Value         0.9993   0.9997   0.9997   0.9997   1.0000
## Prevalence             0.2847   0.1932   0.1746   0.1639   0.1835
## Detection Rate         0.2842   0.1930   0.1744   0.1637   0.1835
## Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
## Balanced Accuracy      0.9989   0.9990   0.9993   0.9991   0.9998
```

Finally we exercise the prediction against the test data targeted as part of the present work, which gives the following results:


```r
# Now get the predictor based on the trained model
rfTestPredictor <- predict(rfModel,targetTestSet[,-length((names(targetTestSet)))])
rfTestPredictor
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

## Conclusions

The results illustrated in the previous section confirms the initial assumption on which for classification problems, the random forest method can be used safely in terms of saving time in terms of feature selection for model training. In this particular case and given the nature of the random forest technique we can safely use as training set by discarding only the incomplete variables. By following that approach we obtained accuracy levels up to 0.9976431 which can be considered as acceptable for the purposes of the present work.

