# Reproducible Research: Peer Assessment 1

PAI_template.Rmd
Project for the Coursera Reproducible Research Class

Prepared by Marcel Merchat
September 20, 2016

The purpose of this project is to provide a reproducible data analysis using
the knitr R package. The project analyzes the steps data taken from a personal
activity monitor at 5 minute intervals in October and November of 2012. 

RAW DATA:
The raw data consists of approxiamtely 17,568 data records of steps taken. Each
record contains three fields:

Column-1: steps
Column-2: date
Column-3: interval - This column indicates the total number of elapsed minutes
          in an indirect way. After every hour or 60 minutes, 40 minutes are
          added so that the minute count changes to 100 instead of 60, 200 after
          2-hours, 300 after three hours and so on until the count is 2355 at
          the end of a day. The next interval reset the count to 0. Although
          this not a time interval in the usual sense, we also call this
          variable "interval" in order to match the raw data.
          
## Loading and preprocessing the steps taken data



```r
library(ggplot2)

file_name <- "activity.csv"

df_steps <-read.csv("activity.csv")

colnames(df_steps) <- c("steps","date","interval")
```



## What is mean total number of steps taken per day?



```r
days <- unique(df_steps[,2])
           
   
day_sums <- tapply(df_steps[,1],df_steps[,2],sum) / 1000
```


```r
hist(day_sums,breaks=c(-0.01,2,4,6,8,10,12,14,16,18,20,22,24),col="blue",
                main="Daily Step Totals",
                xlab = "Thousands of Steps",
                ylab = "Frequency",
                axes = TRUE, ps = 12,
                right=FALSE)
```

![](PA1_template_files/figure-html/hist-1.png) 


Compute the mean and medium number of steps per day.
The median might be zero if the count is zero on most days.

```r
## The mean number of steps
x_bar <- mean(df_steps[,1],na.rm = TRUE)
print("The mean number of steps taken each day is indicated below:")
```

```
## [1] "The mean number of steps taken each day is indicated below:"
```

```r
print(x_bar)
```

```
## [1] 37.3826
```

```r
x_median <- median(df_steps[,1],na.rm = TRUE)
print("The median number of steps taken per is indicated below:")
```

```
## [1] "The median number of steps taken per is indicated below:"
```

```r
print(x_median)
```

```
## [1] 0
```


## What is the average daily activity pattern?



```r
 bad <- is.na(df_steps[,1])
 good <- !bad
 df_steps[,4] <- good 
 colnames(df_steps) <- c("steps","date","interval","good_data")
 df_good <- df_steps[df_steps[,4]==TRUE,]
 interval_totals <- tapply(df_good[,1],df_good[,3],mean)
```



```r
plot(interval_totals,type="l",col="blue",main="Mean Daily Step Totals",
                xlab = "Interval",
                ylab = "Number of Steps",
                ps = 12)
```

![](PA1_template_files/figure-html/lineplot-1.png) 

Which 5-minute interval contains the maximum average number of steps across all
the days in the dataset?


```r
highest_interval <- max(interval_totals) 

biggest_day <- interval_totals[interval_totals==highest_interval]

print("The highest interval was as follows:")
```

```
## [1] "The highest interval was as follows:"
```

```r
print(attributes(highest_interval)$names)
```

```
## NULL
```


## Imputing missing values



```r
missing <- !complete.cases(df_steps)
filled  <- df_steps

print("The number of missing days was as follows:")
```

```
## [1] "The number of missing days was as follows:"
```

```r
print(sum(missing))
```

```
## [1] 2304
```

```r
count <- 1
for(i in seq_along(missing)){
        interv <- df_steps[i,3]
        
        j <- 1 + (df_steps[i,3] - (40 * floor(df_steps[i,3]/100)))/ 5
        
        substitute  <- as.numeric(
                           interval_totals[as.numeric(names(interval_totals))==
                           as.numeric(names(interval_totals[j]))]
                           )[1]
        
        
        if(missing[i]==TRUE){
                filled[i,1] <- substitute
        }
           
}
```


## Histogram with filled data



```r
day_sums2 <- tapply(filled[,1],filled[,2],sum) / 1000

hist(day_sums,breaks=c(-0.01,2,4,6,8,10,12,14,16,18,20,22,24),col="blue",
                main="Daily Step Totals with Fill",
                xlab = "Thousands of Steps",
                ylab = "Frequency",
                axes = TRUE, ps = 12,
                right=FALSE)
```

![](PA1_template_files/figure-html/hist2-1.png) 


## Averages and median for filled data
## Compute the mean and medium number of steps per day.



```r
## The mean number of steps
x_bar2 <- mean(filled[,1],na.rm = TRUE)
print("The mean number of steps taken each day is indicated below:")
```

```
## [1] "The mean number of steps taken each day is indicated below:"
```

```r
print("Mean with Fill")
```

```
## [1] "Mean with Fill"
```

```r
print(x_bar2)
```

```
## [1] 37.3826
```

```r
print("Mean without Fill")
```

```
## [1] "Mean without Fill"
```

```r
print(x_bar)
```

```
## [1] 37.3826
```

```r
x_median2 <- median(filled[,1],na.rm = TRUE)
print("Median with fill:")
```

```
## [1] "Median with fill:"
```

```r
print(x_median2)
```

```
## [1] 0
```

```r
print("Median without fill:")
```

```
## [1] "Median without fill:"
```

```r
print(x_median)
```

```
## [1] 0
```

```r
print("The data indicates that using the average interval value to fill in for
      missing values did not change the mean or the median.")
```

```
## [1] "The data indicates that using the average interval value to fill in for\n      missing values did not change the mean or the median."
```




## Are there differences in activity patterns between weekdays and weekends?




```r
filled[,5] <- weekdays(as.Date(filled[,2]))
filled[,6] <- "NA"
colnames(filled) <- c(order1 =
        "steps","date","interval","good_data","dayofweek","day_cat")

weekly_day_avg <- tapply(filled[,1],filled[,5],mean)
        
## filled[,"daytype"] <- lapply(filled[,"dayofweek"], function(filled) 
##        if(filled[filled[,"dayofweek"]==("Saturday"|"Sunday"),]){
##           filled[filled[,"dayofweek"]==("Saturday"|"Sunday"),]}       
##        else
##        {
##              filled[filled[,"dayofweek"]==("Saturday"|"Sunday"),] <- "Weekday"}      
##        ) 
                
## ggplot(data=df_week_days, aes(wdays2, y=weekly_day_avg)) +  
##        geom_line(colour="blue", linetype="solid", size=1.5) +
##        geom_point(colour="blue", size=4, shape=21, fill="white") +
##        xlab("Day") +
##        ylab("Total Average Steps") +
##        ggtitle("Average Steps by Day of Week")
##        facet_wrap(~ day_cat, ncol = 1)
```
