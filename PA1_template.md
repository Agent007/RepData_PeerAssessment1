# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Remember to set the working directory to be the directory corresponding to the git repository containing this R markdown file.

```r
activity = read.csv(unz("activity.zip", "activity.csv"), header = TRUE, stringsAsFactors = FALSE, 
    colClasses = c("integer", "Date", "integer"))
```


## What is mean total number of steps taken per day?
I. Histogram of total steps per day


```r
library(data.table)
datatable = data.table(na.omit(activity))
stepsperday = datatable[, list(total = sum(steps)), by = date]
totalstepsperday = stepsperday$total
hist(totalstepsperday, freq = TRUE, xlab = "Total Steps", main = "Total Steps Per Day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 


II. Mean & Median steps per day


```r
mean(totalstepsperday)
```

```
## [1] 10766
```

```r
median(totalstepsperday)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
I. Time series plot of the 5 minute interval (x-axis) & avg. number of steps taken, averaged across all days (y-axis)

```r
avgStepsByInterval = datatable[, list(avgSteps = mean(steps)), by = interval]
avgSteps = avgStepsByInterval$avgSteps
plot(avgStepsByInterval$interval, avgSteps, type = "l")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

II. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avgStepsByInterval[avgStepsByInterval$avgSteps == max(avgSteps), ]
```

```
##    interval avgSteps
## 1:      835    206.2
```


## Imputing missing values
I. Total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(!complete.cases(activity))
```

```
## [1] 2304
```


II. Strategy: Fill in missing values with the average steps for that interval

III. New dataset that is equal to the original dataset but with the missing data filled in.

```r
imputed = merge(activity, avgStepsByInterval, by = "interval")
originalSteps = imputed$steps
imputed$steps[is.na(originalSteps)] = avgStepsByInterval$avgSteps[is.na(originalSteps)]
```


IV. Histogram of the total number of steps taken each day

```r
imputedDataTable <- data.table(imputed)
imputedByDate <- imputedDataTable[, list(total = sum(steps)), by = date]
hist(imputedByDate$total, freq = TRUE, xlab = "Total Steps", main = "Total Steps Per Day (with imputation)")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 


## Are there differences in activity patterns between weekdays and weekends?
I. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
imputed$weekend = as.factor(ifelse(weekdays(imputed$date) %in% c("Monday", "Tuesday", 
    "Wednesday", "Thursday", "Friday"), "weekday", "weekend"))
```


II.Panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
par(mfrow = c(2, 1))
weekendDataTable = data.table(imputed)
imputedAvgSteps = weekendDataTable[, list(avgSteps = mean(steps)), by = c("interval", 
    "weekend")]
plot(imputedAvgSteps[imputedAvgSteps$weekend == "weekend"]$interval, imputedAvgSteps[imputedAvgSteps$weekend == 
    "weekend"]$steps, type = "l", xlab = "interval", ylab = "steps", main = "Weekend")
plot(imputedAvgSteps[imputedAvgSteps$weekend == "weekday"]$interval, imputedAvgSteps[imputedAvgSteps$weekend == 
    "weekday"]$steps, type = "l", xlab = "interval", ylab = "steps", main = "Weekday")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


