---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
library(readr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
act <- read_csv("activity.zip")
```

```
## Parsed with column specification:
## cols(
##   steps = col_double(),
##   date = col_date(format = ""),
##   interval = col_double()
## )
```

```r
act$date <- as.Date(as.character(act$date), "%Y-%m-%d")
act <- act %>% arrange(date, interval)
```

## What is mean total number of steps taken per day?

### 1. Make a histogram of the total number of steps taken each day

```r
hist(sapply(split(act, act$date), function(x) sum(x[, 1], na.rm = T)), 
     ylim = c(0, 40), main = "Total Number of Steps Taken Each Day", 
     xlab = "Number of Steps", ylab = "Frequency", col = "blue", labels = T)
```

![](PA1_Yanfei_Chen_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

### 2. Calculate and report the mean and median total number of steps taken per day

```r
mean(sapply(split(act, act$date), function(x) sum(x[, 1], na.rm = T)))
```

```
## [1] 9354.23
```

```r
median(sapply(split(act, act$date), function(x) sum(x[, 1], na.rm = T)))
```

```
## [1] 10395
```

## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
stp_int <- aggregate(steps ~ interval, act, mean)
with(stp_int, plot(interval, steps, type = "l", col = "red", xlab = "Time", ylab = "Average Steps", 
                   main = "Average Number of Steps Taken Across All Day(5-minute interval)"))
```

![](PA1_Yanfei_Chen_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max(stp_int$steps)
```

```
## [1] 206.1698
```

```r
stp_int$interval[which.max(stp_int$steps)]
```

```
## [1] 835
```

## Imputing missing values

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(act))
```

```
## [1] 2304
```

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
int_mean <- aggregate(steps ~ interval, act, mean)
temp_na <- act[!complete.cases(act), -1]
temp_na <- merge(temp_na, int_mean, by = "interval")[, c(3,2,1)]
temp_na <- temp_na %>% arrange(date, interval)
```

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
act1 <- act
act1[!complete.cases(act), 1] <- temp_na[, 1]
```

### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
hist(sapply(split(act1, act1$date), function(x) sum(x[, 1], na.rm = T)), 
     ylim = c(0, 40), main = "Total Number of Steps Taken Each Day", 
     xlab = "Number of Steps", ylab = "Frequency", col = "red", labels = T)
```

![](PA1_Yanfei_Chen_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
mean(sapply(split(act1, act1$date), function(x) sum(x[, 1], na.rm = T)))
```

```
## [1] 10766.19
```

```r
median(sapply(split(act1, act1$date), function(x) sum(x[, 1], na.rm = T)))
```

```
## [1] 10766.19
```
Imputing missing data tends to increase both the mean and the median of the total daily number of steps.

## Are there differences in activity patterns between weekdays and weekends?

### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
library(ggplot2)
weekday12345 <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
act1$day <- factor((weekdays(act1$date) %in% weekday12345), 
                   levels=c(F, T), labels=c("weekend", "weekday"))
act1_de <- aggregate(steps ~ interval + day, act1, mean)
```

### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
g <- ggplot(act1_de, aes(interval, steps)) + geom_line(aes(color = day)) + facet_grid(day ~ .) + labs(x = "Interval") + labs(y = "Average Number of Steps") + labs(title = "Average Number of Steps Taken in Weekdays and Weekends") + theme_bw()
print(g)
```

![](PA1_Yanfei_Chen_files/figure-html/unnamed-chunk-7-1.png)<!-- -->
