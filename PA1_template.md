# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

First we load the data, it's a csv file so no need to worry about options. 
Then munge the date column to be an R Date object. 


```r
Adata <- read.csv("activity.csv")
Adata$date <- as.Date(Adata$date, "%Y-%m-%d")
```

Get rid of the NA. Makes finding mean and median a litle cleaner. 


```r
useData <- subset(Adata, !is.na(steps))
```

## What is mean total number of steps taken per day?

Create a histogram of the total number of steps taken each day.
First we need at aggregate over all intervals. 

```r
totSteps <- aggregate(steps ~ date, data=useData, FUN="sum")
hist(totSteps$steps)
```

![plot of chunk histogram](./PA1_template_files/figure-html/histogram.png) 

Find the mean and median of the total number of steps per day. 


```r
mean(totSteps$steps)
```

```
## [1] 10766
```

```r
median(totSteps$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

Aggregate to find the average number of steps per interval over all days. Then do the plot.

```r
avgedStepsByInterval <- aggregate(steps ~ interval,data=useData, FUN="mean")
with(avgedStepsByInterval, plot(steps, interval, type="l", xlab="Interval", ylab="Daily average steps", main="Average Daily Steps"))
```

![plot of chunk avgInts](./PA1_template_files/figure-html/avgInts.png) 

Now we want to find the interval that has the highest average number of steps. First find that maximum and then the row that contains that value. 


```r
mSteps <- max(avgedStepsByInterval$steps)
subset(avgedStepsByInterval, steps==mSteps)
```

```
##     interval steps
## 104      835 206.2
```


## Inputing missing values

Calculate and report the total number of missing values in the dataset

```r
nrow(subset(Adata, is.na(steps)))
```

```
## [1] 2304
```

Aggregate to find the average number of steps for each day. This is used later to fill in missing data. 


```r
avgedSteps <- aggregate(steps ~ date,data=useData, FUN="mean")
```

Now we want to replace missing data. This is a little slow. 

```r
start <- TRUE
for (i in 1:nrow(Adata)) {
     ii <- Adata[i,]
     if (is.na(ii[1,1])) {
         sst <- avgedSteps[avgedSteps$date == ii[1,2], 2]
         if (length(sst)>0) {
             rii <- list(steps=avgedSteps[avgedSteps$date == ii[1,2], 2], date=ii[1,2], interval=ii[1,3])
         } else {
             rii <- list(steps=0, date=ii[1,2], interval=ii[1,3])
         }
     } else {
         rii <- ii
     }
     if (start==TRUE) {
         store1 <- data.frame(rii)
         names(store1) <- names(Adata)
         start <- FALSE
     } else {
         store1 <- rbind(store1, rii)
     }
}
```

New mean and median, just to see what happened. 

```r
mean(store1$steps)
```

```
## [1] 32.48
```

```r
median(store1$steps)
```

```
## [1] 0
```

Then the new histogram of the modified data, along with the new mean and median.

```r
totStepsAdj <- aggregate(steps ~ date, data=store1, FUN="sum")
hist(totStepsAdj$steps)
```

![plot of chunk unnamed-chunk-5](./PA1_template_files/figure-html/unnamed-chunk-5.png) 

```r
mean(totStepsAdj$steps)
```

```
## [1] 9354
```

```r
median(totStepsAdj$steps)
```

```
## [1] 10395
```

## Are there differences in activity patterns between weekdays and weekends?

Creating factor of weekdays and weekends and storing it. This function 
is applied to the date to decied if it is a week day or weekend day. 

```r
gg <- function(aDay) {
    if (weekdays(aDay) %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")) {
        "weekday"
    } else {
        "weekend"
    }
}
tts <- sapply(store1$date, FUN=gg)
ttsf <- factor(tts)
store1$ext <- ttsf
```

Now two plots, average step counts for each interval over all days. 

```r
par(mfrow=c(2,1), pin=c(5,3), mar=c(4,4,1,1))
with(aggregate(steps ~ interval, data=subset(store1, ext=="weekday",xlab="Interval", ylab="Daily average steps"), FUN="mean"), plot(interval, steps, type="l",xlab="Interval", ylab="Daily average steps:weekdays", cex.lab=0.7))
with(aggregate(steps ~ interval, data=subset(store1, ext=="weekend"), FUN="mean"), plot(interval, steps, type="l",xlab="Interval", ylab="Daily average steps:weekend", cex.lab=0.7))
```

![plot of chunk unnamed-chunk-7](./PA1_template_files/figure-html/unnamed-chunk-7.png) 

