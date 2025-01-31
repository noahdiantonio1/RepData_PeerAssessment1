---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

In this document, I will be performing a series of analyses of the Activity Monitoring Data dataset.

## Loading and preprocessing the data

First, I will load the data into r. Because the zip file is already in the working directory, I will first unzip it and then read the csv file into r.


```r
currentwd <- getwd()
unzip("activity.zip", exdir = currentwd)
activitydata <- read.csv("activity.csv")
```

I will also load the dplyr and lattice packages, which I will use in these questions.


```r
library(dplyr)
library(lattice)
```

I will also create a version of the dataset with missing values removed, which I will use for the first two questions.


```r
activitydatanona <- activitydata[which(activitydata$steps != "NA"),]
```

## What is mean total number of steps taken per day?

First, I will calculate the total number of steps taken per day.


```r
stepsperday <- activitydatanona %>%
        group_by(date) %>%
        summarise(across(.fns = sum))
```

Next, I will produce a histogram of the total number of steps taken per day.


```r
hist(stepsperday$steps, main = "Total Number of Steps Taken Per Day", xlab = "Steps")
```

![plot of chunk unnamed-chunk-122](figure/unnamed-chunk-122-1.png)

Lastly for this section, I will calculate the mean and median values of the total number of steps taken per day.


```r
## Mean
print(c("Mean:", mean(stepsperday$steps)))
```

```
## [1] "Mean:"            "10766.1886792453"
```

```r
## Median
print(c("Median:", median(stepsperday$steps)))
```

```
## [1] "Median:" "10765"
```

## What is the average daily activity pattern?

First, I will calculate the average number of steps for each time interval.


```r
meanstepsperinterval <- activitydatanona %>%
        group_by(interval) %>%
        summarise(across(.cols = steps, .fns = mean))
```

Next, I will make a time series plot of the 5-minute interval and the average number of steps taken.


```r
plot(meanstepsperinterval$interval, meanstepsperinterval$steps, type = "l", main = "Average Steps Per 5-Minute Time Interval", xlab = "5-Minute Time Interval", ylab = "Average Number of Steps")
```

![plot of chunk unnamed-chunk-125](figure/unnamed-chunk-125-1.png)

Lastly for this section, I will find the time interval for which contains the maximum number of steps.


```r
meanstepsperinterval[which.max(meanstepsperinterval$steps),]
```

```
## # A tibble: 1 x 2
##   interval steps
##      <int> <dbl>
## 1      835  206.
```

Interval 835 is the interval with the maximum number of steps, on average. The average number of steps for that interval is 206 steps.

## Imputing missing values

First, I will calculate and report the total number of missing values in the dataset.


```r
sum(is.na(activitydata$steps))
```

```
## [1] 2304
```

There are 2304 missing values in the dataset.  

Next, I will impute the missing data using the mean value for that interval and create a new data set with the imputed data.


```r
activitydataimputed <- activitydata
for(i in 1:nrow(activitydataimputed)) {
       if (is.na(activitydataimputed$steps[i])) {
               intervalvalue <- activitydataimputed$interval[i]
               intervalmean <- mean(activitydataimputed$steps[activitydataimputed$interval == intervalvalue], na.rm = TRUE)
               activitydataimputed$steps[i] <- intervalmean
       }
}
```

Lastly in this section, I will create a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day.


```r
stepsperdayimputed <- activitydataimputed %>%
        group_by(date) %>%
        summarise(across(.fns = sum))

hist(stepsperdayimputed$steps, main = "Total Number of Steps Taken Per Day (with Imputed Data)", xlab = "Steps")
```

![plot of chunk unnamed-chunk-129](figure/unnamed-chunk-129-1.png)

```r
print(c("Mean:", mean(stepsperdayimputed$steps)))
```

```
## [1] "Mean:"            "10766.1886792453"
```

```r
print(c("Median:", median(stepsperdayimputed$steps)))
```

```
## [1] "Median:"          "10766.1886792453"
```

The values for the mean and median are very similar to the mean and median before missing values were imputed. The only difference is that the median value has moved up slightly to be equal to the mean, which has not changed in value at all. This change likely occured because, as can be seen in the histogram, all of the imputed data clusters around the middle of the graph at a value of approximately 10766. This doesn't change the mean but does affect the median.

## Are there differences in activity patterns between weekdays and weekends?

First, I will add a factor variable to the dataset which specifies whether a given day is a weekday or weekend.


```r
weekdaydata <- activitydataimputed %>%
        mutate(week = weekdays(as.Date(date)))
weekdaydata$week[weekdaydata$week == "Monday" | weekdaydata$week == "Tuesday" | weekdaydata$week == "Wednesday" | weekdaydata$week == "Thursday" | weekdaydata$week == "Friday"] <- "Weekday"
weekdaydata$week[weekdaydata$week == "Saturday" | weekdaydata$week == "Sunday"] <- "Weekend"
weekdaydata$week <- as.factor(weekdaydata$week)
```

Next, I will make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.


```r
weekdayinterval <- weekdaydata %>%
        group_by(interval, week) %>%
        summarise(across(.cols = steps, .fns = mean))

weekdayintervalplot <- xyplot(steps ~ interval | week, data = weekdayinterval, type = "l", main = "Average Steps Taken Per 5-Minute Time Interval,\nWeekdays vs. Weekends", xlab = "5-Minute Time Interval", ylab = "Average Number of Steps")
update(weekdayintervalplot, layout=c(1,2))
```

![plot of chunk unnamed-chunk-131](figure/unnamed-chunk-131-1.png)
