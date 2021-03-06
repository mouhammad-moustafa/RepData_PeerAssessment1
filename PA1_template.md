---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
#### 1. Load the data

```r
unzip("activity.zip", overwrite = TRUE)
data <- read.csv("activity.csv", stringsAsFactors = FALSE)
```

#### 2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
## as.Date: converts charcters to Date using "yyyy-mm-dd" format 
data$date <- as.Date(data$date, format = "%Y-%m-%d")
```



## What is mean total number of steps taken per day?

#### 1. compute the total number of steps per day ignoring the missing values

```r
## 
stepsperday <- aggregate(steps ~ date, data = data, FUN = sum)
```

#### 2. Make a histogram of the total number of steps taken each day

```r
## Make a histogram
hist(stepsperday$steps, col="blue", main = "Total number of steps taken each day", xlab = "Number of steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

#### 3. Calculate and report the mean and median total number of steps taken per day

```r
meansteps <- mean(x = stepsperday$steps)
mediansteps <- median(x = stepsperday$steps)
```


The mean of total number of steps taken per day is 1.0766189 &times; 10<sup>4</sup>

The median of total number of steps taken per day is 10765


## What is the average daily activity pattern?
#### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
## compute the average number of steps taken across all days ignoring the missing values
asteps <- aggregate(steps ~ interval, data = data, FUN = "mean")

## create time column by adding coverting interval to H:m
asteps$time <- paste(asteps$interval%/%100, asteps$interval%%100, sep = ":")

## as.POSIXlt: converts charcters to Time using "H:M" format
asteps$time <- as.POSIXlt(asteps$time, format = "%H:%M")

plot(x = asteps$time, y = asteps$steps, type ="l", xlab = "Time", ylab = "Average number of steps", col = "blue")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
## order average steps across all the days 'asteps' by steps, extract the row of the maximum number of steps (first line)
maxstepsrow <- with(asteps, order(-steps))[1]
## extract the value of interval column for the maxstepsrow 
maxinterval <- asteps[maxstepsrow, "interval"]
```

The 5-minute interval that contain the maximum number of steps , on average across all the days in the dataset, is 835 



## Imputing missing values

#### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
## the sum of na values returns the total number of missing values in the dataset for each column
## the first column steps contains all the missing values
nbna <- sum(is.na(data$steps))
```
The total number of missing values in the dataset is 2304


#### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
## use the mean for that 5-minute interval ignoring the missing values
avsteps <- aggregate(steps ~ interval, data = data, FUN = "mean")
## Merge the aggregate data into the original data as a new imputation column based on interval link
data1 <- merge(x = data, y = avsteps, "interval")
## rename columns
colnames(data1) <- c("interval", "steps", "date", "avsteps")
```

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
## replace missing values by average value
data1$steps[is.na(data1$steps)] <- data1$avsteps[is.na(data1$steps)]
## keep usefull columns
data1 <- data1[, c("interval", "steps", "date")]
```

#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

```r
## compute the total number of steps per day ignoring the missing values
stepsperday1 <- aggregate(steps ~ date, data = data1, FUN = sum)
## Make a histogram
hist(stepsperday1$steps, col="blue", main = "Total number of steps taken each day", xlab = "Number of steps")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 

```r
meansteps1 <- mean(x = stepsperday1$steps)
mediansteps1 <- median(x = stepsperday1$steps)
```
The mean of total number of steps taken per day is 1.0766189 &times; 10<sup>4</sup>

The median of total number of steps taken per day is 1.0766189 &times; 10<sup>4</sup>

The difference of mean of total number of steps taken per day is 0

The difference of median of total number of steps taken per day is 1.1886792

The impact of imputing missing data on the estimates of the total daily number of steps is median and mean are almost the same.

## Are there differences in activity patterns between weekdays and weekends?
#### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
data1$weekdaysf <- as.factor(ifelse(weekdays(data1$date) %in% c("Saturday", "Sunday"), "weekend", "weekday"))
```

#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
library(lattice)
 xyplot(steps ~ interval | weekdaysf, data = data1, type = "l", layout = c(1, 2),
        panel = function(x, y, ...) 
         {
                panel.average(x, y, horizontal = FALSE, col = "blue", ...)
         }, xlab="Interval", ylab = "Average number of steps", scales=list(y=list(tick.number=10, limits = c(-10, 300))))
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 

