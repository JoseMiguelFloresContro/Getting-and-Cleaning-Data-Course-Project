# Getting-and-Cleaning-Data-Course-Project
Getting and Cleaning Data Course Project's Solution
rm(list=ls())
library(reshape2)

# As a first step, we are going to download our file(s).

setwd("C:/Users/usuario/Documents/Coursera/3. Getting And Cleaning Data")

# Step 1

filename<-"getdata-projectfiles-UCI HAR Dataset.zip"

# For simplicity, we are going to store the URL in an object called "URL"

if (!file.exists(filename)){
			URL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
			
# We download the compressed file into our working directory

			download.file(URL,filename)
							}	
if (!file.exists("UCI HAR Dataset")){
			unzip(filename)						
							}

# As a second step, we are going to load the "features.txt" & "activity_labels.txt" files into R and do some manipulations

# Step 2

setwd("C:/Users/usuario/Documents/Coursera/3. Getting And Cleaning Data/UCI HAR Dataset")
features<-read.csv("features.txt",sep="",header=FALSE)
activity_labels<-read.csv("activity_labels.txt",sep="",header=FALSE)

# We check how our data frames look like

str(features)
str(activity_labels)

# 	

features$V2<-as.character(features$V2)
activity_labels$V2<-as.character(activity_labels$V2)

# Now we get those ones we are intereseted on

interestedfeatures1<-grep("(.*mean()|std().*)",features$V2)
interestedfeatures2<-features[grep("(.*mean()|std().*)",features$V2),]
colnamesfeatures<-interestedfeatures2[,2]

# We change the string "-mean" & "-std" to "Mean" and "Std" respectively and we delete the parenthesis of the names of our columns

colnamesfeatures<-gsub("-mean()","Mean",colnamesfeatures)
colnamesfeatures<-gsub("-std()","Standard Deviation",colnamesfeatures)
colnamesfeatures<-gsub("[-()]","",colnamesfeatures)

# As a third step we are going to load the data sets (training and tests)

# Step 3

# Training

setwd("C:/Users/usuario/Documents/Coursera/3. Getting And Cleaning Data/UCI HAR Dataset/train")
train.x<-read.csv("X_train.txt",sep="",header=FALSE)
train.y<-read.csv("Y_train.txt",sep="",header=FALSE)
train.z<-read.csv("subject_train.txt",sep="",header=FALSE)

# We check how our data frames look like

str(train.x)
str(train.y)
str(train.z)

# Test

setwd("C:/Users/usuario/Documents/Coursera/3. Getting And Cleaning Data/UCI HAR Dataset/test")
test.x<-read.csv("X_test.txt",sep="",header=FALSE)
test.y<-read.csv("Y_test.txt",sep="",header=FALSE)
test.z<-read.csv("subject_test.txt",sep="",header=FALSE)

# We check how our data frames look like

str(test.x)
str(test.y)
str(test.z)

# We subset only that data we are interested on 

interestedtrain.x<-train.x[,interestedfeatures1]
interestedtest.x<-test.x[,interestedfeatures1]

# As a fourth step we are going to merge our data sets and name the columns

# Step 4

# We merge our data sets

TrainData<-cbind(train.z,train.y,interestedtrain.x)
TestData<-cbind(test.z,test.y,interestedtest.x)
DATA<-rbind(TrainData,TestData)

# We change the name of our columns

names(DATA)<-c("Subject","Activity",colnamesfeatures)

# We take a look to the first records

head(DATA)

# As a fifth step we transformed the Activity and Subject to factors

#Step 5

DATA$Activity<- factor(DATA$Activity, levels = activity_labels[,1], labels = activity_labels[,2])
DATA$Subject <- as.factor(DATA$Subject)
NEWDATA <- melt(DATA, id=c("Subject","Activity"), na.rm=TRUE)
Averages <- dcast(NEWDATA, formula= Subject + Activity~variable, mean)

# We save/export our tidy data file

write.table(Averages,"tidy.txt",sep="\t")
