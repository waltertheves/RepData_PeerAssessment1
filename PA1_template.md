---
title: "Course Project 1 - Reproducible Research"
author: "Walter Breno Theves"
date: "27/07/2020"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a [Fitbit](http://www.fitbit.com/), [Nike Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or [Jawbone Up](https://jawbone.com/up). These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data. 

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site: 
    
- Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]


The variables included in this dataset are:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)  
- **date**: The date on which the measurement was taken in YYYY-MM-DD format  
- **interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Code and explanation of the process

### Loading and preprocessing the data

The first step of this assignment is to download and read the data:

```{r downloading and reading}
# Downloading the file
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, file.path("./", "data.zip"))
unzip("./data.zip")
path <- file.path("./")
list.files((path), recursive=TRUE)
# Reading the data and transforming the date column into Date type 
activity <- read.csv("activity.csv")
activity$date <- as.Date(activity$date)
```

### What is mean total number of steps taken per day?

The missing values are being ignored for this part

```{r total steps and histogram}
library(ggplot2)
stepsperday <- aggregate(steps ~ date, activity, sum)
# Total steps taken each day is stored in stepsperday
ggplot(data=stepsperday, aes(date, steps)) +
    geom_col() + scale_y_continuous(breaks = seq(0,22000, 2000)) +
    scale_x_date(date_breaks = "4 days", date_labels = "%b %d") +
    ggtitle("Total number of steps taken each day") + xlab("Date") + ylab("Steps")

# Histogram of the total number of steps taken each day
hist((stepsperday$steps), main="Histogram os steps taken each day", xlab="Steps",
     col="blue", breaks=30, xlim = c(0,25000), las=1)
```

The variable `stepsperday` is used again now in order to plot the mean and median of the total number of steps taken in every interval of 5 minutes per day.

```{r mean steps}
# Mean and median of steps taken per day
#stepsperday <- aggregate(steps ~ date, activity, mean)
media <- tapply(activity$steps, activity$date, sum)
mean(media, na.rm=TRUE)
median(media, na.rm=TRUE)
```

## What is the average daily activity pattern?

- Make a time series plot (type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

```{r average daily activity pattern}
# Average daily activity pattern
pattern <- aggregate(steps ~ interval, activity, mean)
p <- ggplot(data=pattern, aes(interval, steps)) + geom_line() +
    scale_x_continuous("Day interval", breaks=seq(min(pattern$interval),max(pattern$interval), 200)) + scale_y_continuous("Average number of steps") + ggtitle("Average number of steps taken by interval")
p
```

- Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r 5-minute interval maximum}
# Maximum daily activity pattern
pattern <- aggregate(steps ~ interval, activity, mean)
maximum <- subset(pattern, pattern$steps== max(pattern$steps))
maximum
```

## Imputing missing values

There are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

- Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r rows with NAs}
# Number of rows with missing values
sum(is.na(activity))
```

- Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

- Create a new dataset that is equal to the original dataset but with the missing data filled in.

The strategy used is to fill each NA steps interval with the average measured for that interval.

```{r filling NAs}
pattern <- aggregate(steps ~ interval, activity, mean)
newactivity <- cbind(activity, pattern$steps)
colnames(newactivity) <- c("steps", "date", "interval", "meansteps")
for (i in 1:nrow(newactivity)) {
    if (is.na(newactivity$steps[i])) {
        newactivity$steps[i] <- newactivity$meansteps[i]
    }
}
# the new dataset with the missing data filled in is stored in newactivity
```

- Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}
library(ggplot2)
stepsperday <- aggregate(steps ~ date, newactivity, sum)
# Total steps taken each day is stored in stepsperday
ggplot(data=stepsperday, aes(date, steps)) +
    geom_col() + scale_y_continuous(breaks = seq(0,22000, 2000)) +
    scale_x_date(date_breaks = "4 days", date_labels = "%b %d") +
    ggtitle("Total number of steps taken each day - Missing values adjusted") + xlab("Date") + ylab("Steps")

# Histogram of the total number of steps taken each day
hist((stepsperday$steps), main="Histogram os steps taken each day - Missing values adjusted", xlab="Steps",
     col="blue", breaks=30, xlim = c(0,25000), las=1)

# Comparison between the original mean and the mean with NA values adjusted
media <- tapply(activity$steps, activity$date, sum)
# Original mean
mean(media, na.rm=TRUE)
newmedia <- tapply(newactivity$steps, newactivity$date, sum)
# Adjusted mean
mean(newmedia)
```

The values of mean and median did not change between the original data and the adjusted data with no missing values. This is due to the fact that the filled in values are in fact the mean values for each interval, and so the mean does not change.

## Are there differences in activity patterns between weekdays and weekends?

- Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{r weekdays and weekends}
# code adding a column to characterize as week or weekend day
for (i in 1:nrow(newactivity)) {
    if (weekdays(newactivity$date[i]) %in% c("sábado", "domingo")) {
        newactivity$weekdaytype[i] <- "weekend"
    }   else {
            newactivity$weekdaytype[i] <- "weekday"
    }
}
```

- Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r}
pattern <- aggregate(steps ~ interval + weekdaytype, newactivity, mean)
p <- ggplot(data=pattern, aes(interval, steps)) + geom_line() +
    facet_grid(weekdaytype ~ .) + scale_x_continuous("Day interval", breaks=seq(min(pattern$interval),max(pattern$interval), 100)) + scale_y_continuous("Average number of steps") + ggtitle("Average number of steps taken by interval")
p
```

Analyzing the plots we can see that during weekdays the person starts the day sooner.