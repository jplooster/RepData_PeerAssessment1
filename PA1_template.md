# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
The data that will be used for this literate programming exercise is a file 
containing 17568 obs of a person's step data (61 days, 288 observations/day). For 
reproducibility reasons I have decided to only use the base R system, so all 
plots are done using the base plotting system and all code used is base code.
The file will be loaded into R as follows:

```r
if (! exists("activity_data")) {
        activity_data <- read.csv("activity.csv")
}
clean_df <- activity_data[!is.na(activity_data$steps), ]
```
For the first part of the analysis I've cleaned the data of any NA values, resulting
in the clean_df data frame.


## What is mean total number of steps taken per day?
First analysis is done by totalling steps for every day in the dataframe and discarding
the days without step values measured. With this cleaned data I plotted a histogram and
calculated the mean and median

```r
total_steps <- tapply(clean_df$steps, clean_df$date, sum)
total_steps_clean <- total_steps[!is.na(total_steps)]
hist(total_steps_clean, breaks = 10, xlab = 'Steps/Day', main = 
             "Histogram of Steps/Day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r
mean(total_steps_clean)
```

```
## [1] 10766
```

```r
median(total_steps_clean)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Next step is creating a time series plot based on the average activity per 5-minute
time interval. First the average activity for every 5-minute interval is calculated,
then this data is used to plot the graph.

```r
avg_steps_per_int <- tapply(clean_df$steps, clean_df$interval, mean)
plot(avg_steps_per_int, xaxt = "n", type = "l", xlab = 'time of day', 
     ylab = 'avg steps/5min interval', main = "Time series plot of 
     avg steps throughout day")
axis(1, labels = c('0:00', '5:00', '10:00', '15:00', '20:00'), 
     at = c(0, 60, 120, 180, 240))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 
From the plot it is clear to see most activity is done in the morning. The 5-minute
interval that contains on average across all the days in the dataset, the 
maximum number of steps is the 835
interval.

## Imputing missing values
There are 2304 rows in the original dataset with 
missing values. The following code replaces these missing values with the average 
value of their respective 5-minute interval.

```r
imputed_data <- activity_data
for (row in 1:nrow(imputed_data)){
             if (is.na(imputed_data$steps[row])) {
                     imputed_data$steps[row] <- 
                             avg_steps_per_int[as.character
                                               (activity_data$interval[row])]
             }
             
}
total_steps_imp <- total_steps2 <- tapply(imputed_data$steps, 
                                          imputed_data$date, sum)
hist(total_steps_imp, breaks = 10, xlab = 'Steps/Day', main = 
             "Histogram of Steps/Day (imputed)")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

```r
mean(total_steps_imp)
```

```
## [1] 10766
```

```r
median(total_steps_imp)
```

```
## [1] 10766
```
As you can see this has no effect on the daily step average, which makes sense given
that the missing values were disregarded prior and now replaced with average values
calculated from the clean, reduced, dataset. 
The median however, does shift, due to the fact that the missing values were not
evenly distributed below and above the mean.
With regards to the histogram the middle bin's frequency increases by 8, which is 
exactly what would be expected given that the 8 'NA days' are now replaced by 8
'average days'

## Are there differences in activity patterns between weekdays and weekends?
First the data is split between weekend and weekday data, from there a few earlier 
steps are repeated to create two time series plots. One for the weekdays, and the
other for the weekends. As mentioned earlier I have decided to stick with the base
plotting system for reproducibility reasons, so a panel plot is produced using 
the par(mfrow) solution, rather than going to lattice or ggplot for a more 
sophisticated plot.

```r
imputed_data$wend <- as.factor(ifelse(weekdays(as.Date(activity_data$date)) %in% c("Saturday","Sunday"), "Weekend", "Weekday")) 
imputed_data_wday <- imputed_data[imputed_data$wend == "Weekday", ]
imputed_data_wend <- imputed_data[imputed_data$wend == "Weekend", ]

avg_steps_wday <- tapply(imputed_data_wday$steps, imputed_data_wday$interval, mean)
avg_steps_wend <- tapply(imputed_data_wend$steps, imputed_data_wend$interval, mean)

par(mfrow=c(2,1))
plot(avg_steps_wday, xaxt = "n", type = "l", xlab = "time of day", ylab = "avg steps/5min interval", main = "Weekdays")
axis(1, labels = c('0:00', '5:00', '10:00', '15:00', '20:00'), at = c(0, 60, 120, 180, 240))
plot(avg_steps_wend, xaxt = "n", type = "l", xlab = "time of day", ylab = "avg steps/5min interval", main = "Weekends")
axis(1, labels = c('0:00', '5:00', '10:00', '15:00', '20:00'), at = c(0, 60, 120, 180, 240))
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

From the plots it is rather clear there are distinct differences. On average, peak 
activity during the week is higher, but on the weekends activity throughout the day
is higher and more dispered.
