Assignment 1
========================================================

This is an R Markdown document. Markdown is a simple formatting syntax for authoring web pages (click the **Help** toolbar button for more details on using R Markdown).

When you click the **Knit HTML** button a web page will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

# LOAD AND PREPROCESS THE DATA
## Zipped file downloaded and saved in working directory

```r
data1 <- read.table(unz("activity.zip", "activity.csv"), header=T, sep=",")
data1$date <- as.Date(data1$date,format="%Y-%m-%d")
```

## WHAT IS MEAN TOTAL NUMBER OF STEPS TAKEN PER DAY?
### Create histogram of total number of steps per day, calculate/report mean and median

```r
daily <- tapply(data1$steps, format(data1$date, '%m-%d-%Y'),sum)
hist(daily)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r
mean(daily, na.rm=TRUE)
```

```
## [1] 10766
```

```r
median(daily, na.rm=TRUE)
```

```
## [1] 10765
```

## WHAT IS THE AVERAGE DAILY ACTIVITY PATTERN
### Plot mean number of steps on 5-minute intervals
### Determine which 5-minute interval contains the maximum number of steps

```r
tx = aggregate(data1$steps, list(data1$interval), mean, na.rm=TRUE)
plot(tx, type="l", main="Mean Steps by 5-Minute Interval", ylab="Steps",
     xlab="Interval")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
max(tx$x)
```

```
## [1] 206.2
```

```r
tx[which.max(tx$x),]
```

```
##     Group.1     x
## 104     835 206.2
```

## IMPUTING MISSING VALUES
### Calculate and report the number of rows with missing values

```r
nrow(data1) - nrow(na.omit(data1))
```

```
## [1] 2304
```

### Strategy:  Replace missing data with the mean of the 5-minute interval 
### Create new dataframe with missing data filled in

```r
agg <- aggregate(data1$steps, by=list(data1$interval), FUN=mean, na.rm=TRUE)
data1$mean <- ifelse(data1$interval==agg$Group.1, agg$x)
data2 <- data1
data2$steps <- ifelse(is.na(data2$steps), data2$mean, data2$steps)
```

### Histogram of total number of steps per day and calculate mean and median
####    The histogram, mean and median of total number of steps per day are not
####    affected much by imputing the missing data. 
####    Histogram has the same shape, but lightly higher max frequency.
####    The mean and median are both 10766 as compared to  
####    10766 and 10765, respectively for the original data.

```r
daily2 <- tapply(data2$steps, format(data2$date, '%m-%d-%Y'),sum)
hist(daily2)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

```r
mean(daily2, na.rm=TRUE)
```

```
## [1] 10766
```

```r
median(daily2, na.rm=TRUE)
```

```
## [1] 10766
```

## ARE THERE DIFFERENCES IN ACTIVITY PATTERNS BETWEEN WEEKDAYS AND WEEKENDS?
### Activity patterns on weekdays vs weekends
####     Yes, there are different activity patterns.  Weekday starts earlier and spikes
####     in the morning. Weekend is more consistent across the waking hours.

```r
data2$day <- as.POSIXlt(data2$date)$wday
data2$wkday <- ifelse(data2$day == 0 | data2$day == 6, "weekend", "weekday")
tx2 <- aggregate(data2$steps, list(data2$wkday,data2$interval), mean, na.rm=TRUE)

library(lattice)
xyplot(x~Group.2 | Group.1, data=tx2, type="l",
       layout=c(1, 2), xlab="Interval", ylab="Number of Steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 
