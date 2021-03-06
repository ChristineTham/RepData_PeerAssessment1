# Reproducible Research: Peer Assessment 1

This report is a submission to **Peer Assessment 1** in the **Reproducible Research** course on Coursera which is part of the *Data Science* Specialization.

## Loading and preprocessing the data

### 1. Load the data
First we load the data using `read.csv()`

Note that the data is zipped in the file `activity.zip`, so we call the `unz()`function to unzip and extract the csv file `activity.csv` within the zip file. The result is a data frame with 3 variables.

We use the `colClasses` parameter to set the classes for each column in the data frame.


```r
data <- read.csv(unz("activity.zip", "activity.csv"),
                 colClasses = c("integer", "character", "integer"))
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

### 2. Process/transform the data

We create two extra columns corresponding to the `date` column expressed as a date (`date_asDate`) and the interval converted to time in hours (`time_hour`)


```r
data$date_asDate <- as.Date(data$date, "%Y-%m-%d")
data$time_hour <- (data$interval %/% 100) + (data$interval %% 100) / 60
str(data)
```

```
## 'data.frame':	17568 obs. of  5 variables:
##  $ steps      : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date       : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval   : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ date_asDate: Date, format: "2012-10-01" "2012-10-01" ...
##  $ time_hour  : num  0 0.0833 0.1667 0.25 0.3333 ...
```


## What is mean total number of steps taken per day?

### 1. Calculate the total number of steps taken per day

We calculate the total steps per day by using the `aggregate()` function.


```r
total_steps <- aggregate(data[,"steps"],
                         by = list(data$date),
                         FUN = sum,
                         na.rm = TRUE)
```

### 2. Histogram of the total number of steps taken each day

For simplicity we use the base plotting system to plot a simple histogram of `steps_per_day`. with breaks corresponding to every 1000 steps.


```r
hist(total_steps$x,
     breaks = range(total_steps$x)[2] %/% 1000,
     main = "Histogram of total number of steps per day",
     xlab = "Total number of steps per day", col = "gray")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

### 3. Mean and Median of the total number of steps taken per day

Mean and Median can be calculated using R functions. The results are displayed by rounding to the nearest whole number.


```r
sprintf("Mean steps per day : %.0f steps", mean(total_steps$x))
```

```
## [1] "Mean steps per day : 9354 steps"
```

```r
sprintf("Median steps per day : %.0f steps", median(total_steps$x))
```

```
## [1] "Median steps per day : 10395 steps"
```

## What is the average daily activity pattern?

### 1. Average number of steps taken for each 5 minute interval (averaged across all days)

We generate the average steps per interval using the `aggregate()` function.


```r
avg_steps <- aggregate(data[,c("time_hour", "steps")],
                       by = list(data$interval),
                       FUN = mean,
                       na.rm = TRUE)
```

### 2. Time series plot

We use the `lattice` library to plot a time-series of `avg_steps`


```r
library(lattice)
xyplot(steps ~ time_hour, avg_steps,
       type = "l",
       xlim = c(0,24),
       main = "Average daily activity pattern",
       xlab = "Time (hour)",
       ylab = "Average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

### 3. 5 minute interval that contains the maximum number of steps

This can be derived by reordering `avg_steps` by decreasing `steps` column, and displaying the first row using `head()`

This is interval 0835 (ie. 8:35AM) which has an average of 206 steps.


```r
head(avg_steps[order(avg_steps$steps, decreasing = TRUE),], 1)
```

```
##     Group.1 time_hour    steps
## 104     835  8.583333 206.1698
```

## Imputing missing values

### 1. Total number of missing values in dataset

This can be computed by counting number of rows in the data with `steps` equal `NA`


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

### 2. Strategy for filling in missing values

There are 4 suggested strategies for filling in missing values:

1. mean for that day
2. mean for that 5-minute interval
3. median for that day
4. median for that 5-minute interval

The 4th strategy is selected. We first build a vector `median_steps` containing the median steps for each interval


```r
median_steps <- sapply(split(data, as.factor(data$interval)),
                       function(i) median(i$steps,na.rm = TRUE))
```

### 3. New dataset with missing data filled in

We create a new dataset called `filled_data` and create an extra column called `median_steps` containing the appropriate median for that interval, then we replace all `NA` values with the median.


```r
filled_data <- data
filled_data$median_steps <- median_steps[as.character(data$interval)]
filled_data[is.na(data$steps),"steps"] <- filled_data[is.na(data$steps),"median_steps"]
```


### 4. Revised Histogram of the total number of steps taken each day

We calculate a revised `filled_steps_per_day` using `filled_data` and then plot the revised histogram.


```r
filled_total_steps <- aggregate(filled_data[,"steps"],
                         by = list(filled_data$date),
                         FUN = sum,
                         na.rm = TRUE)
hist(filled_total_steps$x,
     breaks = range(filled_total_steps$x)[2] %/% 1000,
     main = "Revised Histogram of total number of steps per day",
     xlab = "Total number of steps per day", col = "gray")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 

### 5. Revised Mean and Median of the total number of steps taken per day

The revised Mean and Median are calculated using `filled_steps_per_day`.


```r
sprintf("Mean steps per day : %.0f steps", mean(filled_total_steps$x))
```

```
## [1] "Mean steps per day : 9504 steps"
```

```r
sprintf("Median steps per day : %.0f steps", median(filled_total_steps$x))
```

```
## [1] "Median steps per day : 10395 steps"
```

Note that the mean steps per day has increased due to the filling in of data, but the median has not. This is expected, as our strategy for filling in missing data was to use the median steps for the interval.

## Are there differences in activity patterns between weekdays and weekends?

### 1. New factor variable in the dataset to represent "weekday" and "weekend" 

We create a new column called `day_type` in `data` which is a factor to represent whether the date is a weekday or weekend.


```r
day_type = c(Sunday = "weekend",
             Monday = "weekday",
             Tuesday = "weekday",
             Wednesday = "weekday",
             Thursday = "weekday",
             Friday = "weekday",
             Saturday = "weekend")
filled_data$day_type <- as.factor(day_type[weekdays(filled_data$date_asDate)])
```

### 2. Panel plot

We create a new aggregate of `filled_data` grouping by `interval` and `day_type`


```r
avg_steps_bydaytype <- aggregate(filled_data[,c("time_hour", "steps")],
               by = list(filled_data$interval, filled_data$day_type),
               FUN = mean,
               na.rm = TRUE)
```

We then plot the two time series


```r
xyplot(steps ~ time_hour | Group.2,
       avg_steps_bydaytype,
       type = "l",
       xlim = c(0,24),
       layout = c(1,2),
       main = "Average daily activity pattern (weekday/weekend comparison)",
       xlab = "Time (hour)",
       ylab = "Average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png) 

From the above two plots, it would seem the daily activity patterns tends to be

1. Higher on weekdays compared to weekends for intervals ranging from 05:00 to 12:00
2. Higher on weekends for intervals ranging from 12:00 to 20:00

A potential hypothesis for the difference is that the individual is more active in the mornings for weekdays, but more active in the afternoons/evenings for weekends.
