---
Codebook : Getting and Cleaning data project
output: codebook.md
---

## Preparation 


```r
#############    PREPARARTION     ###############
# 1. set work directory
# 2. include all necessary packages
# 3. download all files and dataset
# 4. Unzip data files

setwd("/Users/xinsui/Desktop/Rtraining/GandCData")
path <- getwd()
library(data.table)
library(reshape2)


fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
destfile <- "dataset.zip"
download.file(fileURL, file.path(path, destfile))
unzip(file.path(path,destfile), files = NULL, list = FALSE, overwrite = TRUE,junkpaths = FALSE, exdir = ".", unzip = "internal",
      setTimes = FALSE )
datapath = file.path(path,"/UCI HAR Dataset")
list.files(datapath, recursive = TRUE)
```

```
##  [1] "activity_labels.txt"                         
##  [2] "features_info.txt"                           
##  [3] "features.txt"                                
##  [4] "README.txt"                                  
##  [5] "test/Inertial Signals/body_acc_x_test.txt"   
##  [6] "test/Inertial Signals/body_acc_y_test.txt"   
##  [7] "test/Inertial Signals/body_acc_z_test.txt"   
##  [8] "test/Inertial Signals/body_gyro_x_test.txt"  
##  [9] "test/Inertial Signals/body_gyro_y_test.txt"  
## [10] "test/Inertial Signals/body_gyro_z_test.txt"  
## [11] "test/Inertial Signals/total_acc_x_test.txt"  
## [12] "test/Inertial Signals/total_acc_y_test.txt"  
## [13] "test/Inertial Signals/total_acc_z_test.txt"  
## [14] "test/subject_test.txt"                       
## [15] "test/X_test.txt"                             
## [16] "test/y_test.txt"                             
## [17] "train/Inertial Signals/body_acc_x_train.txt" 
## [18] "train/Inertial Signals/body_acc_y_train.txt" 
## [19] "train/Inertial Signals/body_acc_z_train.txt" 
## [20] "train/Inertial Signals/body_gyro_x_train.txt"
## [21] "train/Inertial Signals/body_gyro_y_train.txt"
## [22] "train/Inertial Signals/body_gyro_z_train.txt"
## [23] "train/Inertial Signals/total_acc_x_train.txt"
## [24] "train/Inertial Signals/total_acc_y_train.txt"
## [25] "train/Inertial Signals/total_acc_z_train.txt"
## [26] "train/subject_train.txt"                     
## [27] "train/X_train.txt"                           
## [28] "train/y_train.txt"
```

## Reading Data from raw files

### 1.Activity labels


```r
# Activity Labels 
activityLabels <- fread(file.path(datapath, "activity_labels.txt"))
setnames (activityLabels, c("V1", "V2"), c("act_num", "Activity"))
activityLabels
```

```
##    act_num           Activity
## 1:       1            WALKING
## 2:       2   WALKING_UPSTAIRS
## 3:       3 WALKING_DOWNSTAIRS
## 4:       4            SITTING
## 5:       5           STANDING
## 6:       6             LAYING
```

### 2.Feature names list


```r
features <- fread(file.path(datapath, "features.txt"))
setnames(features, c("V1", "V2"), c("feature_num", "feature_name"))

# Get only the data on mean and std. dev features
features <- features[grepl("mean\\(\\)|std\\(\\)", feature_name)]
features$feature_num <- features[, paste0("V", feature_num)]
wanted <- features$feature_num
head(features)
```

```
##    feature_num      feature_name
## 1:          V1 tBodyAcc-mean()-X
## 2:          V2 tBodyAcc-mean()-Y
## 3:          V3 tBodyAcc-mean()-Z
## 4:          V4  tBodyAcc-std()-X
## 5:          V5  tBodyAcc-std()-Y
## 6:          V6  tBodyAcc-std()-Z
```

### 3.Read all training and test files

```r
# Read Train and Test Data
dtSubTrain <- fread(file.path(datapath, "train", "Subject_train.txt"))
dtSubTest <- fread(file.path(datapath, "test", "Subject_test.txt"))

dtTrain <- fread(file.path(datapath, "train", "X_train.txt"), select = wanted)
dtTest <- fread(file.path(datapath, "test", "X_test.txt"), select = wanted)
dtActTrain <- fread(file.path(datapath, "train", "Y_train.txt"))
dtActTest <- fread(file.path(datapath, "test", "Y_test.txt"))  

# Make descriptive variable names 
setnames(dtSubTrain, "V1", "subject_id")
setnames(dtActTrain, "V1", "act_num")
setnames(dtSubTest, "V1", "subject_id")
setnames(dtActTest, "V1", "act_num")
```

### 4.Merge training and test data 

```r
dtTrain <- cbind(dtTrain, dtSubTrain, dtActTrain)
dtTest <- cbind(dtTest, dtSubTest, dtActTest)
dtData = rbind(dtTrain, dtTest)
setkey(dtData, subject_id, act_num)

head(dtData)
```

```
##           V1           V2          V3         V4          V5         V6
## 1: 0.2820216 -0.037696218 -0.13489730 -0.3282802 -0.13715339 -0.1890859
## 2: 0.2558408 -0.064550029 -0.09518634 -0.2292069  0.01650608 -0.2603109
## 3: 0.2548672  0.003814723 -0.12365809 -0.2751579  0.01307987 -0.2843713
## 4: 0.3433705 -0.014446221 -0.16737697 -0.2299235  0.17391077 -0.2133875
## 5: 0.2762397 -0.029638413 -0.14261631 -0.2265769  0.16428792 -0.1225450
## 6: 0.2554682  0.021219063 -0.04894943 -0.2245370  0.02231294 -0.1131962
##          V41        V42         V43        V44        V45        V46
## 1: 0.9453028 -0.2459414 -0.03216478 -0.9840476 -0.9289281 -0.9325598
## 2: 0.9411130 -0.2520352 -0.03288345 -0.9839625 -0.9174993 -0.9490782
## 3: 0.9463639 -0.2642781 -0.02557507 -0.9628101 -0.9561309 -0.9719092
## 4: 0.9524451 -0.2598379 -0.02613106 -0.9811001 -0.9643989 -0.9643039
## 5: 0.9471251 -0.2571003 -0.02842261 -0.9769275 -0.9885960 -0.9604447
## 6: 0.9457488 -0.2547778 -0.02652145 -0.9853150 -0.9801945 -0.9662646
##           V81        V82          V83        V84         V85        V86
## 1: -0.1564857 -0.1428530 -0.113078690 -0.1837594 -0.17046131 -0.6138299
## 2: -0.2075541  0.3578428 -0.452400930 -0.1083503 -0.01869285 -0.5475588
## 3:  0.2016045  0.4170823  0.139078170 -0.1776946 -0.02960064 -0.5795071
## 4:  0.3360845 -0.4641436 -0.005025745 -0.1204862  0.02865963 -0.5214649
## 5: -0.2356234 -0.1117772  0.172654600 -0.1924335  0.05398133 -0.4693241
## 6:  0.1159299  0.2346673  0.361505180 -0.2457770 -0.02056663 -0.4659302
##            V121        V122        V123       V124        V125       V126
## 1: -0.479729520  0.08203403 0.256443090 -0.3235458 -0.14193972 -0.4565980
## 2:  0.094091481 -0.30915291 0.086441165 -0.3992529 -0.08841570 -0.4021575
## 3:  0.211200570 -0.27290542 0.101986010 -0.4454378 -0.06308333 -0.3470558
## 4:  0.096081738 -0.16339425 0.025859464 -0.3604054  0.04233342 -0.2761384
## 5:  0.008742388  0.01166058 0.004174515 -0.3775575  0.13371503 -0.3081481
## 6: -0.042556600  0.09761780 0.084655454 -0.5108548  0.02642284 -0.3724244
##           V161         V162        V163       V164       V165       V166
## 1:  0.09424803 -0.476210050 -0.14213364 -0.3457161 -0.4867495 -0.4215080
## 2:  0.16674262 -0.033796125 -0.08926024 -0.2498919 -0.4537442 -0.3698131
## 3: -0.16322550 -0.005560408 -0.23155479 -0.2642317 -0.4246765 -0.3425422
## 4: -0.05462885  0.340289290 -0.26967159 -0.1020531 -0.2434422 -0.3115771
## 5: -0.07566824  0.171466880  0.13645072 -0.1290674 -0.1901072 -0.4183491
## 6: -0.33244254 -0.406247560  0.23877062 -0.2875010 -0.2924124 -0.4825550
##           V201       V202        V214       V215       V227       V228
## 1: -0.22455962 -0.2379807 -0.22455962 -0.2379807 -0.2894243 -0.1650001
## 2: -0.12650269 -0.2133903 -0.12650269 -0.2133903 -0.1385012 -0.1985903
## 3: -0.16010001 -0.2575711 -0.16010001 -0.2575711 -0.1943548 -0.2199436
## 4: -0.07351308 -0.1951145 -0.07351308 -0.1951145 -0.1294801 -0.1739346
## 5: -0.04949205 -0.2110254 -0.04949205 -0.2110254 -0.1598686 -0.1498507
## 6: -0.07739443 -0.2377672 -0.07739443 -0.2377672 -0.2060086 -0.1992724
##           V240        V241       V253       V254       V266        V267
## 1: -0.03439560 -0.16818626 -0.4661497 -0.4336540 -0.2609049 -0.12256680
## 2: -0.14093823 -0.21605518 -0.3899198 -0.4389841 -0.1511153 -0.02904997
## 3: -0.09459356 -0.29084739 -0.3741507 -0.4180319 -0.2304074  0.02542685
## 4: -0.04934062 -0.09012390 -0.2364741 -0.2294418 -0.1513229  0.19526720
## 5: -0.02141046 -0.04463632 -0.2200966 -0.2127722 -0.2258036  0.11028848
## 6: -0.13887531 -0.16730755 -0.3038356 -0.3744300 -0.2904287  0.05782228
##          V268       V269        V270       V271       V345        V346
## 1: -0.3312160 -0.3567070 -0.19956719 -0.1777802 -0.2104645 -0.26352811
## 2: -0.2573071 -0.2621973 -0.02385785 -0.3221639 -0.1783384 -0.12083878
## 3: -0.3773113 -0.2935223 -0.05769317 -0.2900854 -0.1926535 -0.10961071
## 4: -0.3212387 -0.2631256  0.08785532 -0.2169750 -0.1834189 -0.02597198
## 5: -0.2048832 -0.2268023  0.11880106 -0.1463515 -0.2852402 -0.01110185
## 6: -0.2483574 -0.1999707 -0.06209912 -0.1106583 -0.2980468 -0.05172677
##          V347       V348         V349       V350       V424        V425
## 1: -0.5357091 -0.2282532 -0.124274450 -0.6984362 -0.1847807 -0.19802441
## 2: -0.4989475 -0.1140450  0.027847600 -0.5945946 -0.2045095 -0.24583137
## 3: -0.5256478 -0.2358945 -0.005815575 -0.6328668 -0.3170815 -0.20815880
## 4: -0.4874227 -0.1322793  0.020367181 -0.5528494 -0.1622106  0.02655303
## 5: -0.4258950 -0.1692272  0.055776797 -0.5102109 -0.2371058  0.04721125
## 6: -0.4334865 -0.2575384 -0.052800462 -0.4954739 -0.3475347 -0.03515961
##          V426       V427         V428       V429        V503       V504
## 1: -0.3075584 -0.3680772 -0.115047260 -0.5653109 -0.16681083 -0.3995829
## 2: -0.3111780 -0.4613169 -0.009837662 -0.4898550 -0.07927762 -0.4230300
## 3: -0.1857984 -0.4863059  0.009726873 -0.4693567 -0.15631258 -0.4368583
## 4: -0.1804687 -0.4234905  0.044652222 -0.3765651 -0.10437689 -0.3762153
## 5: -0.2579581 -0.4223431  0.176016240 -0.3885971 -0.12319532 -0.3878596
## 6: -0.3338457 -0.5629958  0.055512057 -0.4428802 -0.20002501 -0.3781722
##          V516       V517         V529       V530       V542       V543
## 1: -0.1540448 -0.1846900 -0.222176040 -0.2736495 -0.4318317 -0.4763701
## 2: -0.1784456 -0.2306563 -0.268279820 -0.3146234 -0.4281859 -0.4928844
## 3: -0.1494380 -0.3212563 -0.308670720 -0.4014002 -0.4010383 -0.4819242
## 4: -0.1322222 -0.2326118 -0.060131490 -0.2746461 -0.2176688 -0.2992263
## 5: -0.1160875 -0.2010365 -0.003821466 -0.2462486 -0.1875509 -0.3003380
## 6: -0.1590210 -0.2578159 -0.174531260 -0.3073559 -0.3383588 -0.4650149
##    subject_id act_num
## 1:          1       1
## 2:          1       1
## 3:          1       1
## 4:          1       1
## 5:          1       1
## 6:          1       1
```

### 5.descriptive activity names to name the activities in the data set

```r
dtData <- merge(dtData, activityLabels, by = "act_num", all.x = TRUE)
setkey(dtData, subject_id, act_num, Activity)
```
### Appropriately labels the data set with descriptive variable names.

```r
# Get descriptive feature names.
dtMelted <- data.table(melt(dtData, key(dtData), variable.name = "feature_num"))
dtMelted <- merge(dtMelted, features[, list(feature_num, feature_name)], by = "feature_num", 
            all.x = TRUE)

# Separate different feature variables 

## Features with 1 category
dtMelted$Jerk <- factor(grepl("Jerk",dtMelted$feature_name), labels = c(NA, "Jerk"))
dtMelted$Magnitude <- factor(grepl("Mag",dtMelted$feature_name), labels = c(NA, "Magnitude"))
## Features with 2 categories
n <- 2
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepl("^t", dtMelted$feature_name), grepl("^f",dtMelted$feature_name)), ncol = nrow(y))
dtMelted$Domain <- factor(x %*% y, labels = c("time", "frequency"))
x <- matrix(c(grepl("Acc", dtMelted$feature_name), grepl("Gyro",dtMelted$feature_name)), ncol = nrow(y))
dtMelted$Signals <- factor(x %*% y, labels = c("Accelerometer", "Gyroscope"))
x <- matrix(c(grepl("BodyAcc", dtMelted$feature_name), grepl("GravityAcc", dtMelted$feature_name)), ncol = nrow(y))
dtMelted$Motion <- factor(x %*% y, labels = c(NA, "Body", "Gravity"))
x <- matrix(c(grepl("mean()",dtMelted$feature_name), grepl("std()",dtMelted$feature_name)), ncol = nrow(y))
dtMelted$Variable <- factor(x %*% y, labels = c("Mean", "SD"))

## Features with 3 categories
n <- 3
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepl("-X",dtMelted$feature_name), grepl("-Y",dtMelted$feature_name), grepl("-Z",dtMelted$feature_name)), ncol = nrow(y))
dtMelted$Axis <- factor(x %*% y, labels = c(NA, "X", "Y", "Z"))

head(dtMelted)
```

```
##    feature_num subject_id act_num Activity     value      feature_name
## 1:          V1          1       1  WALKING 0.2820216 tBodyAcc-mean()-X
## 2:          V1          1       1  WALKING 0.2558408 tBodyAcc-mean()-X
## 3:          V1          1       1  WALKING 0.2548672 tBodyAcc-mean()-X
## 4:          V1          1       1  WALKING 0.3433705 tBodyAcc-mean()-X
## 5:          V1          1       1  WALKING 0.2762397 tBodyAcc-mean()-X
## 6:          V1          1       1  WALKING 0.2554682 tBodyAcc-mean()-X
##    Jerk Magnitude Domain       Signals Motion Variable Axis
## 1:   NA        NA   time Accelerometer   Body     Mean    X
## 2:   NA        NA   time Accelerometer   Body     Mean    X
## 3:   NA        NA   time Accelerometer   Body     Mean    X
## 4:   NA        NA   time Accelerometer   Body     Mean    X
## 5:   NA        NA   time Accelerometer   Body     Mean    X
## 6:   NA        NA   time Accelerometer   Body     Mean    X
```
### create tidy data set with the average of each variable for each activity and each subject.

```r
setkey(dtMelted, subject_id, Activity, Domain, Signals, Motion, 
       Jerk, Magnitude, Axis, Variable)
dtTidy <- dtMelted[, list( average = mean(value)), by = key(dtMelted)]
```


### write into a text file

```r
write.table(dtTidy, "tidy.txt", row.names = FALSE, quote = FALSE)
```
