# Reproducible Research: Peer Assessment 1
==========================================
Created by Xiaodan Zhang on July 18, 2014

### Basic settings
```{r}
echo = TRUE  # Always make code visible
options(scipen = 1)  # Turn off scientific notations for numbers
```
### Loading and processing the data
```{r}
unzip("activity.zip")
data <- read.csv("activity.csv", colClasses = c("integer", "Date", "factor"))
attach(data)
data$month <- as.numeric(format(date, "%m"))
noNA <- na.omit(data)
rownames(noNA) <- 1:nrow(noNA)
head(noNA)
dim(noNA)
library(ggplot2)
```

### What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Make a histogram of the total number of steps taken each day
```{r}
attach(noNA)
ggplot(noNA, aes(date, steps)) + geom_bar(stat = "identity", colour = "steelblue", fill = "steelblue", width = 0.7) + facet_grid(. ~ month, scales = "free") + labs(title = "Histogram of Total Number of Steps Taken Each Day", x = "Date", y = "Total number of steps")
```

2. Calculate and report the mean and median total number of steps taken per day

Mean total number of steps taken per day:
```{r}
totalSteps <- aggregate(steps, list(Date = date), FUN = "sum")$x
mean(totalSteps)
```
Median total number of steps taken per day:
```{r}
median(totalSteps)
```

### What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r}
avgSteps <- aggregate(steps, list(interval = as.numeric(as.character(interval))), FUN = "mean")

ggplot(avgSteps, aes(interval, y = x)) + geom_line(color = "steelblue", size = 0.8) + labs(title = "Time Series Plot of the 5-minute Interval", x = "5-minute intervals", y = "Average Number of Steps Taken")
```

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
attach(avgSteps)
avgSteps[x == max(x), ]
```

### Imputing missing values
1. The total number of rows with NAs:

```{r}
sum(is.na(data))
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

My strategy is to use the mean for that 5-minute interval to fill each NA value in the steps column.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
newData <- data 
for (i in 1:nrow(newData)) {
    if (is.na(newData$steps[i])) {
        newData$steps[i] <- avgSteps[which(newData$interval[i] == avgSteps$interval), ]$x
    }
}

head(newData)
sum(is.na(newData))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

```{r}
ggplot(newData, aes(date, steps)) + geom_bar(stat = "identity", colour = "steelblue", fill = "steelblue", width = 0.7) + facet_grid(. ~ month, scales = "free") + labs(title = "Histogram of Total Number of Steps Taken Each Day (no missing data)", x = "Date", y = "Total number of steps")
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Mean total number of steps taken per day:
```{r}
attach(newData)
newTotalSteps <- aggregate(steps, list(Date = date), FUN = "sum")$x
newMean <- mean(newTotalSteps)
newMean
```
Median total number of steps taken per day:
```{r}
newMedian <- median(newTotalSteps)
newMedian
```
Compare them with the two before imputing missing data:
```{r}
oldMean <- mean(totalSteps)
oldMedian <- median(totalSteps)
newMean - oldMean
newMedian - oldMedian
```
So, after imputing the missing data, the new mean of total steps taken per day is the same as that of the old mean; the new median of total steps taken per day is greater than that of the old median.

### Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r}
head(newData)
newData$weekdays <- factor(format(newData$date, "%A"))
levels(newData$weekdays)
levels(newData$weekdays) <- list(weekday = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"), weekend = c("Saturday", "Sunday"))
levels(newData$weekdays)
table(newData$weekdays)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r}
attach(newData)
avgSteps <- aggregate(steps, list(interval = as.numeric(as.character(interval)), weekdays = weekdays), FUN = "mean")
names(avgSteps)[3] <- "meanOfSteps"
attach(avgSteps)
library(lattice)
xyplot(meanOfSteps ~ interval | weekdays, layout = c(1, 2), type = "l", xlab = "Interval", ylab = "Number of steps")
```
