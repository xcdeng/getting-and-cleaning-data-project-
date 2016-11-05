library(data.table)
library(reshape2)
library(plyr)

#set directory for raw data after save and unzip the file
setwd("C:/Users/xc/data science/R program assignment/data clearning/Week 4/UCI HAR Dataset")

#identify columns with mean and std only
activity<-t(read.table("features.txt"))
activityWanted<-grep(".*mean.*|.*std.*", activity[2,])

#use descriptive activity names to name the activities 
colName<-activity[2,][activityWanted]
#colName<-t(as.matrix(colName))
colName=gsub('-mean', 'Mean',colName)
colName=gsub('-std', 'Std',colName)
colName<-gsub('[-()]', '',colName)

# read test data
testData<-read.table("./test/X_test.txt")[activityWanted]

#add in activity labels + features for test data

label1<-read.table("./test/subject_test.txt")
label2<-read.table("./test/y_test.txt")
label3<-read.table("activity_labels.txt")

mergeTstData<-join(label2,label3)

#mergeTstData=merge(label2,label3,by.x="V1",by.y="V1",all=TRUE)
#test data with activitiy label and feasutres

testData<-cbind(label1,mergeTstData$V2,testData)
colnames(testData)<-c("subject","activity",colName)
write.table(testData, "testData.txt", row.names = FALSE, quote = FALSE)

#read train data
trainData<-read.table("./train/X_train.txt")[activityWanted]

#add in activity labels + features for train data

label4<-read.table("./train/subject_train.txt")
label5<-read.table("./train/y_train.txt")
mergeTratData<-join(label5,label3)
#test data with activitiy label and feasutres
trainData<-cbind(label4,mergeTratData$V2,trainData)
colnames(trainData)<-c("subject","activity",colName)



#combine data and add labels
allData <- rbind(trainData, testData) 



#aggregation (mean) based on subject and activity

allData.melt<- melt(allData, id = c("subject", "activity"))
allData.mean <- dcast(allData.melt, subject + activity ~ variable, mean)
write.table(allData.mean, "tidy.txt", row.names = FALSE, quote = FALSE)

