---
title: "Machine Learning to Determine Bicep Curl Method"
author: "Jennifer Fadimba"
date: "Sunday, March 26, 2017"
output: html_document
---

I used a subset of data (https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv) from the http://groupware.les.inf.puc-rio.br/har#dataset#ixzz4cSvunXAe  to predict classe, which range from A to E and correspond to various ways of performing bicep curls.

```{r message=FALSE}
train1<-read.csv("C:/Users/Owner/Documents/pml-training.csv")
library(caret)
```

The starting dataset has 19622 observations and 160 variables.
```{r}
dim(train1)
```

##Sifting Out Variables
I first reduced the number of columns in the dataset by removing columns with near zero variances.  I took out all the columns with NA values because each of them had 19,216 NA values.  Due to each of these previously mentioned columns only having numerical values for two percent of the observations, I decided to omit them as predictors.

```{r}
nearzero<-nearZeroVar(train1)
train2<-train1[,-nearzero]

findna<-colSums(sapply(train2, is.na))
head(findna[findna>0])
train2<-train2[,findna==0]
```

I also got rid of the first five columns which dealt with observation number (X), username, and timestamps  as well as num_window (the sixth column).  As shown in the plot below, num_window increases similarly as observation number goes up within each classe.
```{r}
qplot(X, num_window, data = train2, color=classe)
train2<-train2[,-c(1:6)]
```

Now 53 variables remain.

##Creating the Model

I split 75 percent of the observations into a training set and the rest into a validation set.  
```{r}
set.seed(325)
inTrain<-createDataPartition(train2$classe, p=0.75, list=FALSE)
traindata<-train2[inTrain,]
validdata<-train2[-inTrain,]
```

A random forest algorithm was used to predict classe using the remaining 52 variables because it is one of the most accurate algorithm.  However, I used cross validation with 10 subsamples within the trainControl attribute of the train function to avoid overfitting.
```{r message=FALSE, cache=TRUE}
set.seed(325)
modelFit<-train(classe~., data=traindata, method="rf", trControl= trainControl(method="cv", number=10, allowParallel = TRUE))
```

##Predicting

The model was used to predict the classe of the observations in the validation set.  The Accuracy of the model on the validation data is 99.55%, therefore I estimate out-of-sample error to be a low 0.45%.

```{r}
predmod<-predict(modelFit, validdata)
confusionMatrix(predmod, validdata$classe)
```


Data source:  Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.