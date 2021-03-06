# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
The data was downloaded directly from the course website, unzipped, and read into R by performing the following code:

```r
require(stats)
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", temp)
unzip(temp, "activity.csv")
data <- read.csv("activity.csv")
unlink(temp)
```

This code creates a temporary file and downloads the zip file into it before decompressing it and reading the activity.csv file into R.

## What is mean total number of steps taken per day?
The number of steps per day were calculated using R's tapply function, which creates an array wherein each sum can be accessed by the date.

```r
daily.steps <- tapply(data$steps, data$date, sum)
```

This new array, which I called daily.steps, was then used to generate a histogram.

```r
h <- hist(daily.steps, main="Steps per Day", xlab="Number of steps per day", col="gray")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

The daily.steps array was also used to calculate the mean and median number of steps per day.

```r
mean(daily.steps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(daily.steps, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
In order to determine the average daily activity pattern, the data was first cleaned to remove NA values. Then, an array of the average numebr of steps per five minute interval was created.

```r
clean <- data[complete.cases(data$steps),]
interval.steps <- tapply(clean$steps, clean$interval, mean)
```

This new array was then used to create a time series plot with the base package plotting system.

```r
plot(names(interval.steps), interval.steps, type="l", col="red",
     xlab="Intervals", ylab="Average Number of Steps Taken",
     main="Average Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

From the graph, it looks like the most steps on average are taken around the 800 to 900 intervals. To determine precisely which interval had the most steps on average, the which.max function was applied to the array.

```r
which.max(interval.steps)
```

```
## 835 
## 104
```

The 835 interval features the most steps on average, with 104 steps.

## Imputing missing values
In investigating the data, it appeared that NAs were only occuring in the with measures of the "steps" variable. Out of curiosity and to confirm this, I took a comprehensive count of NAs in the data set by subtracting the rows in a complete.cases-filtered version of the dataset from the full set. Then, more specifically, I produced a table of NAs in the "steps" variable measures. The number of rows with missing values was 2304, which is equal to the number of NAs within the "steps" measure.

```r
nrow(data)-nrow(data[complete.cases(data),])
```

```
## [1] 2304
```

```r
table(is.na(data$steps))
```

```
## 
## FALSE  TRUE 
## 15264  2304
```

I copied the data set to a new variable I called "complete."" I used the array containing average number of steps per interval to fill in the missing intervals and complete the data set. This means that the "complete" data set contains the average values of steps per interval in all of the missing "steps" measures.

```r
complete <- data
complete$steps[is.na(complete$steps)] <- interval.steps[as.character(complete$interval[is.na(complete$steps)])]
```

To demonstrate that there are no missing values in the complete set, here's a summary of the original data set followed by a summary of the complete set.

```r
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

```r
summary(complete)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 27.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##                   (Other)   :15840
```

As in the first step, I created an array of sums of steps per day and used this to generate a histogram and to find the mean and median total steps per day.

```r
complete.daily <- tapply(complete$steps, complete$date, sum)
hist(complete.daily, main="Steps per Day (no NAs)", xlab="Total Steps per Day", col="gray")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

The mean and the median are both 10766.19 total steps per day. 

```r
mean(complete.daily)
```

```
## [1] 10766.19
```

```r
median(complete.daily)
```

```
## [1] 10766.19
```

This is equal to the mean of total steps per day found using the original dataset, which makes sense considering that the complete set had measures for eachc interval equal to the average steps taken in that interval across all days. The median shifted up from 10765. This also makes sense--the median of the original set would have been a whole number, as steps per interval measures were taken in whole steps. The upward shift from the orignal to the complete set is due to the previously unavailable values now being populated with averages.

## Are there differences in activity patterns between weekdays and weekends?
In order to process the data to learn whether there are differences between weekdays and weekends, I wanted to use the mutate function from the dplyr package. For this plot, I wanted to work with a different graphic package, ggplot, so I imported these first. Then, to determine whether a measure was taken on a weekday or weekend, I added a column to my complete data set called "days." Using the weekdays function on the date variable, I populated this column with days as factors. To mutate these specific days into weekend or weekday factors, I created a vector, "weekends," with the strings "Saturday" and "Sunday", and then mutated the "days" column with "weekend" or "weekday" depending on whether the factor in the "days" column was found in the "weekends" vector.

```r
require(dplyr)
```

```
## Loading required package: dplyr
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
complete$days <- as.factor(weekdays(as.Date(levels(complete$date))))
weekends <- c("Saturday", "Sunday")
complete <- mutate(complete, days = as.factor(ifelse(days %in% weekends, "weekend", "weekday")))
```

To plot the mean number of steps taken during each interval on a weekend and weekday, I first used the aggregate function to create a dataset, "agg", including the days ("weekday" or "weekend" factors), intervals, and averages. 

This made it very easy to create a panel plot with the ggplot package. In my call to ggplot, I used facet_grid to tell R how to split the average steps per interval into the two plots.

```r
require(ggplot2)
```

```
## Loading required package: ggplot2
```

```r
agg <- aggregate(complete$steps, by=list(complete$days, complete$interval), mean)
names(agg) <- c("days", "interval", "steps")
gg <- ggplot(agg, aes(interval, steps)) + geom_line() + ggtitle("Average number of steps per interval") + labs(x="Interval", y="Number of Steps") + facet_grid(days~.)
print(gg)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

There are similarities in the weekday and weekend time series--peaks and valleys happen around the same times. However, on the weekends, there are more exaggerated differences between the peaks and valleys than during the weekdays. In fact, average steps taken per interval exceed 250 on the weekends, and more freqeuently exceed 150 in the afternoons than weekdays.
