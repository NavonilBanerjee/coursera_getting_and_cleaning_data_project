filename<-"getdata_dataset.zip"
##Download & unzip the dataset
if(!file.exists(filename)){
  fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip "
  download.file(fileURL,filename)
}## if not been able to download from R manually download the zip file in R working directory.
if(!file.exists("UCI HAR Dataset")){
  unzip(filename)
}

#Load activity labels+features
activityLabels<-read.table("UCI HAR Dataset/activity_labels.txt")
features<-read.table("UCI HAR Dataset/features.txt")

#Changing factors into character
activityLabels[,2]<-as.character(activityLabels[,2])
features[,2]<-as.character(features[,2])

#identify features with mean & std
featureswanted<-grep("mean|std",features[,2])
featureswanted_names<-features[featureswanted,2]
featureswanted_names=gsub('-mean','Mean',featureswanted_names)
featureswanted_names=gsub('-std','Std',featureswanted_names)
featureswanted_names=gsub('[-()]','',featureswanted_names)


##Load all datasets
##all train datasets
train <- read.table("UCI HAR Dataset/train/X_train.txt")
train<-train[,featureswanted] ## considering obesrvations for only mean & std
trainActivities <- read.table("UCI HAR Dataset/train/Y_train.txt")
trainSubjects <- read.table("UCI HAR Dataset/train/subject_train.txt")
train <- cbind(trainSubjects, trainActivities, train)
##all test datasets
test <- read.table("UCI HAR Dataset/test/X_test.txt")
test<-test[,featureswanted]
testActivities <- read.table("UCI HAR Dataset/test/Y_test.txt")
testSubjects <- read.table("UCI HAR Dataset/test/subject_test.txt")
test <- cbind(testSubjects, testActivities, test)

# merge datasets and add labels
allData <- rbind(train, test)
names(allData)<-c("Subject","Activity",featureswanted_names)

#Proper naming & factorization of activities
allData$Activity<-factor(allData$Activity,levels = activityLabels[,1],labels = activityLabels[,2])

#subject into factor
allData$Subject<-as.factor(allData$Subject)

##Calculating groupwise means
#Calling Library plyr to use ddply function
library(plyr)
tidy_data<- ddply(allData, c("Subject","Activity"), numcolwise(mean))

write.table(tidy_data,row.names = FALSE )
