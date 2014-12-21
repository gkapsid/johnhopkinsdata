johnhopkinsdata
===============

from Coursera - John Hopkins courses
My complete script with comments
For the script to work properly it is necessary to install a few libraries like:
downloader and dplyr
It consist of only one script

# install library downloader if necessary, else manually download and unzip
library(downloader) 
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download(url, "getCleanProj.zip", mode="wb")
unzip("getCleanProj.zip")
unlink(url)

# read test and train data and merge
x_test<-read.table("UCI HAR Dataset/test/X_test.txt", header=FALSE)
x_train<-read.table("UCI HAR Dataset/train/X_train.txt", header=FALSE)
x_merged<-rbind(x_train, x_test)

# read subject files for test and train data and merge
sub_test<-read.table("UCI HAR Dataset/test/subject_test.txt", header=FALSE)
sub_train<-read.table("UCI HAR Dataset/train/subject_train.txt", header=FALSE)
sub_merged<-rbind(sub_train, sub_test)

# read activity files for test and train data and merge
y_test<-read.table("UCI HAR Dataset/test/y_test.txt", header=FALSE)
y_train<-read.table("UCI HAR Dataset/train/y_train.txt", header=FALSE)
y_merged<-rbind(y_train, y_test)

# merge x data, subjects and activities
mergedcol<-cbind(x_merged, sub_merged, y_merged)

# name the cols according the features.txt file
colnamesM<-read.table("UCI HAR Dataset/features.txt", header=F)
colnames(mergedcol)<-colnamesM[,2]
# name subject and activities cols
names(mergedcol)[562:563]<-c("subject", "activityNo")

# select cols with "mean and std"

#Give meaningful description to activities
library(dplyr) # install library if necessary
# remove duplicates as they don't contain mean or std
dupcols<-duplicated(names(mergedcol))
mergedcol<-mergedcol[!dupcols]

#Give meaningful names to activities
mergedcol<-mutate(mergedcol, activity=ifelse(activityNo==1, "WALKING", ifelse(activityNo==2,"WALKING_UPSTAIRS",ifelse(activityNo==3,"WALKING_DOWNSTAIRS",ifelse(activityNo==4,"SITTING",ifelse(activityNo==5, "STANDING",ifelse(activityNo==6, "LAYING",-1)))))))



merged2<-tbl_df(mergedcol)

# select cols with "mean" in name
mergedselmean<-select(merged2, contains("mean", ignore.case=TRUE)) 
# select cols with "std" in name
mergedselstd<-select(merged2, contains("std", ignore.case=TRUE))
# colbind the selected cols
mergedfin<-cbind(mergedselmean, mergedselstd)
# colbind the subject col
mergedfin<-cbind(merged2[478], mergedfin) 
# colbind the activities col 
mergedfin<-cbind(mergedfin, merged2[480])


# remove dots, spaces, etc from col names
names(mergedfin)<-sub("...","",names(mergedfin),fixed=TRUE)
names(mergedfin)<-sub("..","",names(mergedfin),fixed=TRUE)
names(mergedfin)<-sub(".","",names(mergedfin),fixed=TRUE)
names(mergedfin)<-sub(".","",names(mergedfin),fixed=TRUE)
names(mergedfin)<-sub("mean","Mean",names(mergedfin),fixed=TRUE)
names(mergedfin)<-sub("std","Std",names(mergedfin),fixed=TRUE)


# final step
dat_avg <- aggregate(mergedfin[,3:87],list(mergedfin$activity,mergedfin$subject),mean)

#give cols 1:2 meaningful names
names(dat_avg)[1:2]<-c("ByActivity", "BySubject")

#write result in file
write.table(dat_avg, "getDataProj.txt")
