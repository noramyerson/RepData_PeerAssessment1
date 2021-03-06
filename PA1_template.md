#Reproducible Research 
###Peer Review Assignment 1, Week 2
*By Nora Myerson*

Overview:  This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data can be found here: [Activity Monitoring Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

+ **steps:** Number of steps taking in a 5-minute interval (missing values are coded as NA)
+ **date:** The date on which the measurement was taken in YYYY-MM-DD format
+ **interval:** Identifier for the 5-minute interval in which measurement was taken  

The dataset is stored in a comma-separated-value (CSV) file and there are a total o44f 17,568 observations in this dataset.

###Loading and Pre-Processing the Data
Be sure you download the document to your current working directory. Next, load in necessary packages:

```r
require(dplyr) #structuring data
require(ggplot2) #creating visualizations
options(scipen=1, digits=1) ##rounding results
```

Below, we will read the data into variable 'active', put it into a table, and look at the dimensions and a summary of the data.

```r
active <- read.csv(file = "activity.csv")
active <- tbl_df(active)
dim(active)
```

```
## [1] 17568     3
```

```r
summary(active)
```

```
##      steps              date          interval   
##  Min.   :  0    2012-10-01:  288   Min.   :   0  
##  1st Qu.:  0    2012-10-02:  288   1st Qu.: 589  
##  Median :  0    2012-10-03:  288   Median :1178  
##  Mean   : 37    2012-10-04:  288   Mean   :1178  
##  3rd Qu.: 12    2012-10-05:  288   3rd Qu.:1766  
##  Max.   :806    2012-10-06:  288   Max.   :2355  
##  NA's   :2304   (Other)   :15840
```
Based on the above, we can confirm that all 17,568 records and 3 dimensions are present in our data. We also see that there are 2,304 NA values in steps that will be addressed in the 'Imputing Missing Values' section of this analysis. 

###What is the mean total number of steps taken per day?

####1. First we will make a histogram of the total number of steps taken per day by creating an array of the sum of steps for each day in the data and creating a plot with the ggplot2 package. We will exclude missing values for this exercise. 

```r
dailySteps <- aggregate(steps~date, active, sum)
ggplot(dailySteps, aes(x = steps))+ geom_histogram(fill = "blue", binwidth = 1000)+ labs(title = "Steps per Day: Histogram", x="Number of Steps per Day",y = "Frequency")+theme(plot.title = element_text(hjust = 0.5))
```

![plot of chunk stepsHist](figure/stepsHist-1.png)

####2. Next we will calculate the **mean** and **median** of steps taken per day. We will exclude missing values for this exercise. 

```r
meanSteps <- mean(dailySteps$steps, na.rm= TRUE)
medianSteps <- median(dailySteps$steps, na.rm= TRUE)
```
For the number of steps taken per dat the mean is **10766.2** and the median is **10765**.

###What is the average daily activity pattern?
####1. We will make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days. 
First we calculate the average number of steps taken for each 5 minute interval.

```r
intervalSteps <- aggregate(steps ~ interval, data = active, FUN = mean)
names(intervalSteps) <- c("interval","AvgSteps") #givethe dataframe meaningful names
```
Next we will plot the data.

```r
plot(x= intervalSteps$interval, y = intervalSteps$AvgSteps, type = "l",main = "Average Steps per Day by 5 Minute Intervals: Across All Days",xlab = "5 Min Intervals", ylab = "Average Steps Taken Across All Days")
```

![plot of chunk IntervalPlot](figure/IntervalPlot-1.png)

####2. Which 5-minute interval, on average, across all the days in the dataset, contains the maximum number of steps?

```r
maxInterval <- which.max(intervalSteps$AvgSteps)
maxSteps <- intervalSteps[maxInterval,]
maxSteps
```

```
##     interval AvgSteps
## 104      835      206
```
The 5 minute interval with the maximum number of average steps is interval **835** with **206.2** steps on average.

###Imputing Missing Values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

####1. Calculate and report the total number of missing values in the dataset.

```r
numberNA <- sum(is.na(active$steps))
```
There are **2304** missing values in the dataset.

####2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
Given that the data is broken into 5 minute intervals, the average value for the intervals with missing data will be used to replace the NA values. This is a reasonable approach with the data. 

####3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activeImpute <- active #create copy to preserve original data
#merge active data with intervalSteps to bring in the average steps for each interval as a distinct column.
activeMerge <- merge(activeImpute, intervalSteps, by = "interval")
#if a value in steps is NA, we'll take the corresponding average steps value for that interval
activeImpute$steps  <- ifelse(is.na(activeMerge$steps),activeMerge$AvgSteps,activeMerge$steps)
#check that we eliminated NAs in our activeImpute dataset
numberNACheck <- sum(is.na(activeImpute$steps))
```
Per our check, the number of NA values after we imputed missing data was **0** proving we successfully addressed the missing values. 

####4. Make a histogram of the total number of steps taken each day. 

```r
dailyStepsImputed <- aggregate(steps~date, activeImpute, sum)
ggplot(dailyStepsImputed, aes(x = steps))+ geom_histogram(breaks=seq(1000, 55000, by = 1000),fill = "blue")+ labs(title = "Steps per Day - Imputed Data: Histogram", x="Number of Steps per Day",y = "Frequency")+theme(plot.title = element_text(hjust = 0.5))
```

![plot of chunk HistImputed](figure/HistImputed-1.png)

####Calculate and report the mean and median total number of steps taken per day. 

```r
meanStepsImputed <- mean(dailyStepsImputed$steps)
medianStepsImputed <- median(dailyStepsImputed$steps)
##create a table to compare values visually
original <- c(meanSteps,medianSteps)
imputed <- c(meanStepsImputed, medianStepsImputed)
difference <- c(meanSteps-meanStepsImputed, medianSteps-medianStepsImputed)
compare <- data.frame(value = c("mean", "median"),original, imputed, difference)
compare
```

```
##    value original imputed difference
## 1   mean    10766   10766          0
## 2 median    10765   10352        413
```

####Do these values differ from the estimates from the first part of the assignment? 
The dataframe above shows that our mean has stayed the same our original and imputed means have a difference of **0**, at **10766.2**.
On the other hand our median has changed from **10765** to **10351.6**, shifting down by **413.4** steps . The imputation of the average steps by interval has caused a decrease in the median value, but has not impacted the average steps per day.

####What is the impact of imputing missing data on the estimates of the total daily number of steps?
Since we imputed the data with the average step value by interval, there was no significant impact. Hence the difference of 0 between the original and imputed mean steps. 

###Are there differences in activity patterns between weekdays and weekends?

####1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
#create a new data  set to preserve changes
activeWeek <- activeImpute
#add a column to represent the day of the week as a factor
activeWeek$dayType <- as.factor(ifelse(weekdays(as.Date(activeWeek$date)) %in% c("Saturday","Sunday"),"weekend","weekday"))
#Check levels
levels(activeWeek$dayType)
```

```
## [1] "weekday" "weekend"
```

####2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
#aggregate data for average steps per interval for each level of dayType
dailyStepsWeek <- aggregate(steps~interval+ dayType,data = activeWeek, FUN=mean)
##plot it out
ggplot(dailyStepsWeek, aes(interval, steps))+ geom_line(color = "blue")+facet_grid(dayType~.)+xlab("5 min Interval")+ylab("Average Steps")+labs(title ="Average Steps per 5 Min Interval by Day Type")+theme(plot.title = element_text(hjust = 0.5))
```

![plot of chunk finalPlot](figure/finalPlot-1.png)







