---
title: "Getting and Cleaning Data - Course Project Assignment"
output: github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## GitHub Documents

This is the course project for Getting and Cleaning Data Coursera course.

The appended R script, run_analysis.R, conducts the following:

- Download the dataset from web if it does not already exist in the working directory.
- Read both the train and test datasets and merge them into x(measurements), y(activity) and subject, respectively.
- Load the data(x's) feature, activity info and extract columns named 'mean'(-mean) and 'standard'(-std).
- Modify column names to descriptive. (-mean to Mean, -std to Std, and remove symbols like -, (, ))
- Extract data by selected columns(from step 3), and merge x, y(activity) and subject data. Also, replace y(activity) column to it's name by refering activity label (loaded step 3).
- Generate 'Tidy Dataset' that consists of the average (mean) of each variable for each subject and each activity. The result is shown in the file tidy_dataset.txt.

## Including Code

You can include R code in the document as follows:

```{run_analysis.R}

#1. get dataset from web
rawDataDir <- "C:/Users/cheryljeanne/Documents/Git/ProgrammingAssignment3"
rawDataUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
rawDataDFn <- paste(rawDataDir, "/", "rawData.zip", sep = "")
dataDir <- "./data"

if (file.exists(rawDataDir)) {
  download.file(url = rawDataUrl, destfile = rawDataDFn)
}
if (file.exists(rawDataDFn)) {
  unzip(zipfile = rawDataDFn, exdir = dataDir)
}


#2. merge {train, test} data set
# refer: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones
# train data
x_train <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/train/X_train.txt"))
y_train <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/train/Y_train.txt"))
s_train <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/train/subject_train.txt"))

# test data
x_test <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/test/X_test.txt"))
y_test <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/test/Y_test.txt"))
s_test <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/test/subject_test.txt"))

# merge {train, test} data
x_data <- rbind(x_train, x_test)
y_data <- rbind(y_train, y_test)
s_data <- rbind(s_train, s_test)


#3. load feature & activity info
# feature info
feature <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/features.txt"))

# activity labels
a_label <- read.table(paste(sep = "", dataDir, "/UCI HAR Dataset/activity_labels.txt"))
a_label[,2] <- as.character(a_label[,2])

# extract feature cols & names named 'mean, std'
selectedCols <- grep("(mean|std)", as.character(feature[,2]))
selectedColNames <- feature[selectedCols, 2]
selectedColNames <- gsub("-mean", "Mean", selectedColNames)
selectedColNames <- gsub("-std", "Std", selectedColNames)
selectedColNames <- gsub("[-()]", "", selectedColNames)


#4. extract data by cols & using descriptive name
x_data <- x_data[selectedCols]
allData <- cbind(s_data, y_data, x_data)
colnames(allData) <- c("Subject", "Activity", selectedColNames)

allData$Activity <- factor(allData$Activity, levels = a_label[,1], labels = a_label[,2])
allData$Subject <- as.factor(allData$Subject)


#5. generate tidy data set
library(reshape2)
meltedData <- melt(allData, id = c("Subject", "Activity"))
tidyData <- dcast(meltedData, Subject + Activity ~ variable, mean)

write.table(tidyData, "./tidy_dataset.txt", row.names = FALSE, quote = FALSE)
```

