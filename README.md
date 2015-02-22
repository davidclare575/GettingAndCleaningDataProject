# GettingAndCleaningDataProject

#the first step is figuring out what variables we want to pull in from the datasets
#the assignment said only extract mean and standard deviations

#pulling in features text file
features <- read.table("features.txt")

#we are going to search through the variables listed and create new columns for whether those are either
#mean or std. We turn the names column into characters, the use grepl to search, creating a new column for both
#mean and std.
features$V2 <- as.character(features$V2)

features$meanvar = ifelse(grepl(c("mean"),features$V2, ignore.case = TRUE), TRUE, FALSE)

features$stdvar = ifelse(grepl(c("std"),features$V2, ignore.case = TRUE), TRUE, FALSE)

#we are going to only keep the variable numbers of those that are either TRUE for mean or std.
#this will leave us with what columns to pull
featurestopull <- features$V1[(features$meanvar == TRUE) | (features$stdvar == TRUE)]
####we are also going to create a similar table that is only means for the tidy dataset later
####meanstopull <- features$V1[(features$meanvar == TRUE)]


#now going to pull in raw xtraindata and then create table with only mean and std features
#using the featurestopull
trainraw <- read.table("X_train.txt")
trainmeanstd <- trainraw[,(featurestopull)]

#now we need to assign activities and subjects to the rows of the xtrain data
trainsubs <- read.table("subject_train.txt")
trainacts <- read.table("y_train.txt")
trainsubsacts <- cbind(trainsubs, trainacts)
trainfull <- cbind(trainsubsacts, trainmeanstd)

#now we do the same thing, pull in raw and then subjects and activities for the test data
testraw <- read.table("X_test.txt")
testmeanstd <- testraw[,(featurestopull)]
testsubs <- read.table("subject_test.txt")
testacts <- read.table("y_test.txt")
testsubacts <- cbind(testsubs, testacts)
testfull <- cbind(testsubacts, testmeanstd)

#now we put the training and testing data together
fullset <- rbind(trainfull, testfull)

#now we are going to label the columns
#first pull names from the features set the same way we picked column numbers
namestopull <- features$V2[(features$meanvar == TRUE) | (features$stdvar == TRUE)]

#then put on the two at the front for subjects and activities
names <- c("SUBJECT", "ACTIVITY", namestopull)

colnames(fullset) <- names

#now we are subbing in labels for activities
#first pull in that table with the numbers and activities
activitynames <- read.table("activity_labels.txt")

#now just use the integer activity in the data set to pick which row to pull a label from

fullset$ACTIVITY <- activitynames[fullset$ACTIVITY, 2]

#at this point we have the full data set, now we have to make it tidy
#we are going to have a set where each row is a subject and activity
#and the values are the average across the multiple measures of that activity
#for the subject.
#every subject should have six rows

#make sure you have plyr ready
library(plyr)
aves <- ddply(fullset, .(SUBJECT, ACTIVITY), function(x) colMeans(x[3:88]))ave

#now write it out to turn in
write.table(aves, "averages.txt", row.name=FALSE)
