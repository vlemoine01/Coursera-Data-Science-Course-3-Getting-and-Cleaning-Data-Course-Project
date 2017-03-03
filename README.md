# Coursera-Data-Science-Course-3-Getting-and-Cleaning-Data-Course-Project

# Getting and Cleaning Data Course Project

The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.
One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:
http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 
Here are the data for the project:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 


## Review Criteria

1.	The submitted data set is tidy. 
2.	The Github repo contains the required scripts.
3.	GitHub contains a code book that modifies and updates the available codebooks with the data to indicate all the variables and summaries calculated, along with units, and any other relevant information.
4.	The README that explains the analysis files is clear and understandable.
5.	The work submitted for this project is the work of the student who submitted it.


## Script to perform analysis run_analysis.R

First need to download, read, and label data.
This code will require two libraries: dplyr, tidyr:

```
library(dplyr)
library(tidyr)
fileurl<-("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip")
download.file(fileurl,destfile="./data.zip")unzip("./data.zip")
```
 ##check out files before reading to see format
##read tables in
```
xtest<-read.table("./UCI HAR Dataset/test/X_test.txt")
ytest<-read.table("./UCI HAR Dataset/test/y_test.txt")
subtest<-read.table("./UCI HAR Dataset/test/subject_test.txt")
xtrain<-read.table("./UCI HAR Dataset/train/X_train.txt")

ytrain<-read.table("./UCI HAR Dataset/train/y_train.txt")
subtrain<-read.table("./UCI HAR Dataset/train/subject_train.txt")
actlabels<-read.table("./UCI HAR Dataset/activity_labels.txt")
xvarlabels<-read.table("./UCI HAR Dataset/features.txt")
```
##assign in column names
```
colnames(xtrain)<-xvarlabels[,2]
colnames(xtest)<-xvarlabels[,2]
colnames(ytrain)<-"activityid"
colnames(ytest)<-"activityid"
colnames(subtest)<-"volunteer"
colnames(subtrain)<-"volunteer"
colnames(actlabels)<-c("activityid","activityname")

```


the R script called run_analysis.R does the following. 
Merges the training and the test sets to create one data set.
```
##merge datasets for test and train
train<-cbind(xtrain,ytrain,subtrain)
test<-cbind(xtest,ytest,subtest)
masterdata<-rbind(train,test)
```
Extracts only the measurements on the mean and standard deviation for each measurement.
Please note that I am excluding variable names that contain “meanFreq” as I interpreted the directions as only wanting the mean() and std() variables defined in the features_info.txt document, and excluded the signal window sample averages used in the angle variable.
```
##extract only mean and std dev activities

subsetin<-grepl("std|mean|volunteer|activityid",names(masterdata))
subsetdata<-masterdata[,subsetin==TRUE]
subsetdata<-select(subsetdata,-contains("meanFreq"))
```
Uses descriptive activity names to name the activities in the data set
```
##add activityname to dataframe by activityid
subsetdata2<-merge(subsetdata,actlabels,by="activityid",all.x=TRUE)
```
Appropriately labels the data set with descriptive variable names. 
```
##creat descriptive names for variables
##need to take out punctuation
subnames<-names(subsetdata2)
subnames<-gsub("[()]","",subnames)
subnames<-gsub("-","",subnames)
subnames<-gsub("BodyBody","Body",subnames)
subnames<-gsub("mean","Mean",subnames)
subnames<-gsub("std","Std",subnames)
subnames<-gsub("^t","time",subnames)
subnames<-gsub("^f","freq",subnames)
colnames(subsetdata2)<-subnames
```
From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```
##create a second data set that contains the average of each variable for each activity by each subject
subgroup<-subsetdata2%>%group_by(volunteer,activityname)
newdata<-summarise_each(subgroup,funs(mean),timeBodyAccMeanX:freqBodyGyroJerkMagStd)
##write newdata to a final tidydata csv file
write.csv(newdata, file = "tidydata.csv")
write.table(newdata, file="tidydata.txt",row.names=FALSE)
```
