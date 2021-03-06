# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Read the orignial csv file into a data frame and load dplyr package for further analysis

```r
require(dplyr)
dfOrig <- read.csv("activity.csv", colClasses = c("integer", "Date", "integer"))
dfOrig <- tbl_df(dfOrig)
```

## What is mean total number of steps taken per day?

Calculate the total number of steps taken each day into a new data frame

```r
dfDay<- dfOrig %>% 
    group_by(date) %>% 
    summarize(totalSteps = sum(steps, na.rm = T))
```

Make a histogram

```r
hist(dfDay$totalSteps, 
     main = "Historgram of total number of steps taken each day",
     xlab = "total steps taken per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

Calculate the mean and the median

```r
mean(dfDay$totalSteps)
```

```
## [1] 9354.23
```

```r
median(dfDay$totalSteps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

Calculate the average number of steps taken by 5-min interval into a new data frame

```r
dfInterval <- dfOrig %>% 
    group_by(interval) %>% 
    summarize(avgSteps = mean(steps, na.rm =T))                      
```

Make a time series plot

```r
with(dfInterval, plot(interval, avgSteps, 
                      type = "l",
                      main = "Average Number of Steps by 5-min Interval",
                      xlab = "5-min interval",
                      ylab = "avg. number of steps each day"))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

Identify the top interval with highest average steps

```r
arrange(dfInterval, desc(avgSteps)) [1,]
```

```
## Source: local data frame [1 x 2]
## 
##   interval avgSteps
## 1      835 206.1698
```
It turns out to be the interval from 8h35 to 8h40.

## Imputing missing values

Find how many NA values that the dataset contains with summary function.

```r
summary(dfOrig)
```

```
##      steps             date               interval     
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0.00   Median :2012-10-31   Median :1177.5  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
##  NA's   :2304
```
2304 rows contain missing values.

As one strategy for filling missing values, I plug in average steps for the 5-min interval as calculated above.  


```r
dfNew <- merge(dfOrig, dfInterval) %>% #combine data together
    tbl_df() %>% #change to data frame tbl format
    mutate(steps = ifelse(is.na(steps), avgSteps, steps)) %>% 
    select(steps, date, interval) # keep orginal format
```

Calculate the total number of steps taken each day

```r
dfDay2<- dfNew %>% 
    group_by(date) %>% 
    summarize(totalSteps = sum(steps, na.rm = T))
```

Make a new histogram

```r
hist(dfDay2$totalSteps, 
     main = "Historgram of total number of steps taken each day",
     xlab = "total steps taken per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

Calculate the new mean and median values

```r
mean(dfDay2$totalSteps)
```

```
## [1] 10766.19
```

```r
median(dfDay2$totalSteps)
```

```
## [1] 10766.19
```

Both the mean and median values turn out to be higher than the original ones, which indicates that imputing the missing data has a moderate inflating effect on the estimates of total daily steps.

## Are there differences in activity patterns between weekdays and weekends?


```r
dfNew <- dfNew %>%
    mutate(dayofWeek = factor(ifelse(weekdays(dfNew$date) %in%
                                            c("Saturday", "Sunday"), 
                                        "weekend", 
                                        "weekday")))
```

Calculate the average number of steps taken by 5-min interval 

```r
dfInterval2 <- dfNew %>% 
    group_by(interval, dayofWeek) %>% 
    summarize(avgSteps = mean(steps, na.rm =T))                      
```

Make a panel plot with lattice system 

```r
require(lattice)
```

```
## Loading required package: lattice
```

```r
with(dfInterval2, xyplot(avgSteps ~ interval | dayofWeek,
                         type = "l",
                         layout = c(1,2),
                         ylab = "Number of steps",
                         main = "Average Number of Steps by 5-min Interval"))
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png) 

The difference between weekdays and weekends is evidently shown: in weekends the steps are relatively taken throughout the daytime while in weekdays the commuting hours, morning and evening, feature the peaks.


# Thank you for viewing my work!
