# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. Clear any old variables and functions from the working space environment.

2. Unzip the data file from the git repository <http://github.com/rdpeng/RepData_PeerAssessment1>

```r
unzip("activity.zip")
#import the data, handle NAs
activity <- read.csv("activity.csv", sep=",", header=TRUE, na.strings=c("NA","NaN", " "))
```
3. convert date column from string values to date class

```r
activityFormatted <- activity
activityFormatted$date <- as.Date(activityFormatted$date, "%Y-%m-%d")
```

## What is mean total number of steps taken per day?
Aggregate data by total steps on a given day, ignoring NAs

```r
totalStepsPerDay <- aggregate(steps ~ date, data = activityFormatted, FUN = sum)
```

1. Generate a histogram of total steps per day

```r
# graph by day
library(ggplot2)
library(scales)

g <- ggplot(data = totalStepsPerDay, aes(x=date, y=steps)) 
g <- g + geom_bar(stat="identity")
g <- g + ggtitle("Histogram of total steps per day")
# Format : month/day
g <- g + scale_x_date(labels = date_format("%b %d")) +
  theme(axis.text.x = element_text(angle=45))
#g <- g + geom_hline(aes(yintercept=meanSteps), colour="#990000", linetype="dashed") 
#+ geom_text(aes(0,meanSteps,label = "Mean number of steps per day", vjust = -1))
#g <- g + geom_hline(aes(yintercept=medianSteps), colour="#009900", linetype="dashed") 
#+ geom_text(aes(0,meanSteps,label = "Median number of steps per day", vjust = -1))
print(g)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

2. Calculate and report the mean and median total number of steps taken per day

```r
meanSteps <- mean(totalStepsPerDay$steps)
medianSteps <- median(totalStepsPerDay$steps)

meanSteps <- round(meanSteps, digits=0)
medianSteps <- round(medianSteps, digits=0)
```
The mean number of steps taken per day are 1.0766\times 10^{4} and the median number of steps are 1.0765\times 10^{4}


## What is the average daily activity pattern?
A time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
#group by mean Steps for Intervals
avDailySteps <- aggregate(steps ~ interval, data = activityFormatted, FUN = mean)

plot(avDailySteps$interval, avDailySteps$steps, type="l", main = "Mean steps per 5 min interval", xlab = "5 min interval", ylab = "Mean number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->



```r
# the maximum was, max(avDailySteps$steps)
#what is the index of the time interval was this maximum value?
maximumIntervalIndex = which(avDailySteps$steps == max(avDailySteps$steps))

#the interval with the maximum steps on average is
maxStep <- avDailySteps$interval[maximumIntervalIndex]
```
The 5-minute interval, from on average across all the days in the dataset, which contains the maximum number of steps is 835.


## Imputing missing values



```r
totalMissingVal <- sum(is.na(activity$steps))
```
1. The total number of missing values in the dataset (i.e. the total number of rows with NAs) is 2304.


2. Where a missing value (NA) was found in the interval column, the NA was replaced with the mean found for all corespondly matching 5 minute intervals.

```r
# where we find a NULL step find its index, use this to find the interval then look up our mean interval values
activity2 <- merge(activity, avDailySteps, by = "interval", suffixes = c("", ".y"))

# Pseudo code
# if activity2$steps == 0 then mod i by length of avDailySteps for index of value to retrieve from avDailySteps


# Create a new dataset that is equal to the original dataset but with the missing data filled in. 
#(we call this activity2)

maxIndex = nrow(avDailySteps)

rm(activity2)

# Create a new dataset that is equal to the original dataset but with the missing data filled in. 
#(we call this activity2)
activity2 <- activityFormatted
count <- 1
head(activity2)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
for (steps in activity2$steps){
  if (is.na(steps)){
    
    index <- (count%%maxIndex)
    
    #as R arrays  are numbered from 1 not 0, when mod equals 0 we have to adjust to max position in arrary
    if(index == 0){
      index <- maxIndex
    }
    # copy the mean step value into the mod Count position in the activity step row (only if NA)
    activity2$steps[count] <- avDailySteps$steps[index]    
  }
  count <- count+1
}
```

3. A histogram of the total number of steps taken each day 

```r
totalStepsPerDay2 <- aggregate(steps ~ date, data = activity2, FUN = sum)

tg <- ggplot(data = totalStepsPerDay2, aes(x=date, y=steps)) 
tg <- tg + geom_bar(stat="identity")
tg <- tg + ggtitle("Histogram of total steps per day")
# Format : month/day
tg <- tg + scale_x_date(labels = date_format("%b %d")) +
  theme(axis.text.x = element_text(angle=45))
#g <- g + geom_hline(aes(yintercept=meanSteps), colour="#990000", linetype="dashed") 
#+ geom_text(aes(0,meanSteps,label = "Mean number of steps per day", vjust = -1))
#g <- g + geom_hline(aes(yintercept=medianSteps), colour="#009900", linetype="dashed") 
#+ geom_text(aes(0,meanSteps,label = "Median number of steps per day", vjust = -1))
print(tg)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
 
4. The calculated mean and median total number of steps taken per day:

```r
meanSteps2 <- mean(totalStepsPerDay2$steps)
medianSteps2 <- median(totalStepsPerDay2$steps)

meanSteps2 <- round(meanSteps2, digits=0)
medianSteps2 <- round(medianSteps2, digits=0)
```
Mean total = 1.0766\times 10^{4}
Median total = 1.0766\times 10^{4}


5. Do these values differ from the estimates from the first part of the assignment? 

```r
print(paste("mean Steps with NA:",meanSteps))
```

```
## [1] "mean Steps with NA: 10766"
```

```r
print(paste("mean Steps without NA:",meanSteps2))
```

```
## [1] "mean Steps without NA: 10766"
```

```r
print(paste("median Steps with NA:",medianSteps))
```

```
## [1] "median Steps with NA: 10765"
```

```r
print(paste("median Steps without NA:",medianSteps))
```

```
## [1] "median Steps without NA: 10765"
```

```r
print(" exact match.")
```

```
## [1] " exact match."
```

6. What is the impact of imputing missing data on the estimates of the total daily number of steps?

The impact to altering the mean and median overall is negligible. However the impact to the data on days with NA values previously, gives an indication of activity where there was none successfully recorded before. 


## Are there differences in activity patterns between weekdays and weekends?

```r
# Using the weekdays() function on the dataset with the filled-in missing values for this part.
weekendOrNot <- function(date) {
  if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
    "weekend"
  } else {
    "weekday"
  }
}

activity2$day <- as.factor(sapply(activity2$date, weekendOrNot))

# A panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) 
# and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

#group by mean Steps for Intervals
avDailyStepsWkDAY <- (subset(activity2, day == "weekday"))[,0:3]
avDailyStepsWkDAY <- aggregate(steps ~ interval, data = avDailyStepsWkDAY, FUN = mean)

avDailyStepsWkEND <- (subset(activity2, day == "weekend"))[,0:3]
avDailyStepsWkEND <- aggregate(steps ~ interval, data = avDailyStepsWkEND, FUN = mean)

#  avDailySteps2 <- aggregate(steps ~ interval | day, data = activity2, FUN = mean)



plot(avDailyStepsWkEND, type="l", main = "Weekend - Mean steps per 5 min interval", xlab = "5 min interval", ylab = "Mean number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
plot(avDailyStepsWkDAY, type="l", main = "Weekday - Mean steps per 5 min interval", xlab = "5 min interval", ylab = "Mean number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

We can see that that for both weekends and weekdays the day starts with a lot of steps and then this reduces a lot for the rest of the day during the week. People appear more active during the day at weekends.

