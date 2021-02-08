---
title: "Assignment 1"
author: "Maria Gilbert"
date: "2/8/2021"
output: md_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


```{r}
library(tidyverse)
library(ggplot2)
library(scales)
library(FNN)
library(class)
library(rsample)
library(caret)
library(modelr)
library(parallel)
library(foreach)
GasPrices <- read.csv("/Volumes/G-DRIVE mobile USB-C/GasPrices.csv")
bikeshare <- read.csv("/Volumes/G-DRIVE mobile USB-C/bikeshare.csv")
ABIA <- read.csv("/Volumes/G-DRIVE mobile USB-C/ABIA.csv")
sclass <- read.csv("/Volumes/G-DRIVE mobile USB-C/sclass.csv")
```


# Data Visualization: gas prices
A) Gas stations charge more if they lack direct competition in sight (boxplot).
```{r, echo=TRUE}
ggplot(data=GasPrices)+geom_boxplot(aes(x=factor(Competitors),
    y=Price))+ggtitle("Direct Competition within sight \n vs. Price")+scale_y_continuous(labels=scales::number_format(accuracy=0.01))+xlab("Competitors within sight")+ylab("Price, USD")
```
\
B) The richer the area, the higher the gas price (scatter plot).
```{r, echo=TRUE}
ggplot(data=GasPrices)+geom_point(mapping=aes(x=Income,y=Price))+scale_y_continuous(labels=scales::number_format(accuracy=0.01))+scale_x_continuous(labels=scales::number_format(accuracy=1000),breaks=seq(0,150000,25000))+ggtitle("Median Household Income within the zip code \n vs. Gas Price")
```
\
C) Shell charges more than other brands (bar plot).
```{r, echo=TRUE}
ggplot(data=GasPrices,aes(x=Brand,y=Price))+geom_bar(stat="summary",fun.y="mean")+scale_y_continuous(labels=scales::number_format(accuracy=0.01),breaks=seq(1.84,1.90,0.01),limits=c(1.84,1.90),oob = rescale_none)+ggtitle("Average price of gas by brand")
```
\
D) Gas stations at stoplights charge more (faceted histogram).
```{r, echo=TRUE}
ggplot(data=GasPrices)+geom_boxplot(aes(x=Brand,y=Price))+facet_wrap(~Stoplight)+ggtitle("Average price of gas by brand, at stoplights and not at stoplights")+scale_y_continuous(labels=scales::number_format(accuracy=0.01))
```
\
E) Gas stations with direct highway access charge more (your choice of plot).
```{r, echo=TRUE}
ggplot(data=GasPrices)+geom_violin(aes(x=factor(Highway),y=Price))+scale_y_continuous(labels=scales::number_format(accuracy=0.01))+ggtitle("Average price of gas with and without direct highway access")+xlab("Direct highway access")+ylab("Price")
```
\

# Data visualization: a bike share network
Plot A: a line graph showing average bike rentals versus hour of the day.
```{r, echo=TRUE}
ggplot(data=bikeshare)+geom_line(aes(hr,total),stat="summary",fun.y="mean")+scale_x_continuous(breaks=0:24)+ggtitle("Average bike rentals by hour of the day")+xlab("Hour")+ylab("Average number of rentals")
```
\
Plot B: a faceted line graph showing average bike rentals versus hour of the day, faceted according to whether it is a working day.
```{r, echo=TRUE}
ggplot(data=bikeshare)+geom_line(aes(hr,total),stat="summary",fun.y="mean")+facet_wrap(~workingday)+ggtitle("Average bike rentals by hour of the day, split between working and non-working days")+xlab("Hour")+ylab("Average number of rentals")
```
\
Plot C: a faceted bar plot showing average ridership during the 8 AM hour by weather situation code, faceted according to whether it is a working day or not.
```{r, echo=TRUE}
bikeshare %>%
  filter(hr=="8") %>% ggplot()+geom_bar(aes(x=weathersit,y=total),stat="identity")+facet_wrap(~workingday)+ggtitle("Average bike rentals between 8 and 9 AM by weather situation code, on working and non-working days")+xlab("Weather situation code")+ylab("Average number of rentals")
```
\

# Data visualization: flights at ABIA
I would like to examine the difference in flight delays between different months, times of day, and carriers. With this information I could choose which month to travel, what time I should try to book my flight for, and which carrier to book my flight with, if my goal is to avoid all kinds of delays. 
```{r, echo=TRUE}
ggplot(data=ABIA,aes(x=factor(Month),y=DepDelay))+geom_bar(stat="summary",fun.y="mean")+scale_x_discrete(labels=c("1"="Jan","2"="Feb","3"="Mar","4"="Apr","5"="May","6"="Jun","7"="Jul","8"="Aug","9"="Sep","10"="Oct","11"="Nov","12"="Dec"))+ggtitle("Average departure delay by month")+xlab("Month")+ylab("Average minutes delayed")
```
\

Next, I will see which carriers have the shortest and longest average departure delays.
```{r, echo=TRUE}
ggplot(data=ABIA,aes(x=factor(UniqueCarrier),y=DepDelay))+geom_bar(stat="summary",fun.y="mean")+ggtitle("Average departure delay by carrier")+xlab("Carrier")+ylab("Average minutes delayed")+scale_x_discrete(labels=c("9E"="Endeavor","AA"="American","B6"="Jetblue","CO"="Shanghai","DL"="Delta","EV"="Envoy","F9"="Frontier","MQ"="Amer Eagle","NW"="Northwest","OH"="PSA","OO"="SkyWest","UA"="United","US"="US Airways","WN"="Southwest","XE"="ExpressJet","YE"="Mesa"))+theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```
\


# K-nearest neighbors
Splitting the 350 AMG data and the 63 AMG data.
\
```{r, echo=TRUE}
threefifty <- sclass[sclass$trim=="350",]
sixtyfive <- sclass[sclass$trim=="65 AMG",]
```
\

## 350 AMG
Splitting the data into a training and a testing set
```{r, echo=TRUE}
ggplot(data=threefifty)+geom_point(mapping=aes(x=mileage,y=price))
threefifty_split = initial_split(threefifty,prop=0.9)
threefifty_train = training(threefifty_split)
threefifty_test = testing(threefifty_split)
```
\

Running K-nearest-neighbors, from K=2 to K=25
```{r, echo=TRUE}
knn2 = knnreg(price~mileage,data=threefifty_train,k=2)
knn3 = knnreg(price~mileage,data=threefifty_train,k=3)
knn4 = knnreg(price~mileage,data=threefifty_train,k=4)
knn5 = knnreg(price~mileage,data=threefifty_train,k=5)
knn6 = knnreg(price~mileage,data=threefifty_train,k=6)
knn7 = knnreg(price~mileage,data=threefifty_train,k=7)
knn8 = knnreg(price~mileage,data=threefifty_train,k=8)
knn9 = knnreg(price~mileage,data=threefifty_train,k=9)
knn10 = knnreg(price~mileage,data=threefifty_train,k=10)
knn11 = knnreg(price~mileage,data=threefifty_train,k=11)
knn12 = knnreg(price~mileage,data=threefifty_train,k=12)
knn13 = knnreg(price~mileage,data=threefifty_train,k=13)
knn14 = knnreg(price~mileage,data=threefifty_train,k=14)
knn15 = knnreg(price~mileage,data=threefifty_train,k=15)
knn16 = knnreg(price~mileage,data=threefifty_train,k=16)
knn17 = knnreg(price~mileage,data=threefifty_train,k=17)
knn18 = knnreg(price~mileage,data=threefifty_train,k=18)
knn19 = knnreg(price~mileage,data=threefifty_train,k=19)
knn20 = knnreg(price~mileage,data=threefifty_train,k=20)
knn21 = knnreg(price~mileage,data=threefifty_train,k=21)
knn22 = knnreg(price~mileage,data=threefifty_train,k=22)
knn23 = knnreg(price~mileage,data=threefifty_train,k=23)
knn24 = knnreg(price~mileage,data=threefifty_train,k=24)
knn25 = knnreg(price~mileage,data=threefifty_train,k=25)
```
\
Calculating the out-of-sample root mean-squared error (RMSE) for each value of k
```{r, echo=TRUE}
rmse2 = rmse(knn2,threefifty_test)
rmse3 = rmse(knn3,threefifty_test)
rmse4 = rmse(knn4,threefifty_test)
rmse5 = rmse(knn5,threefifty_test)
rmse6 = rmse(knn6,threefifty_test)
rmse7 = rmse(knn7,threefifty_test)
rmse8 = rmse(knn8,threefifty_test)
rmse9 = rmse(knn9,threefifty_test)
rmse10 = rmse(knn10,threefifty_test)
rmse11 = rmse(knn11,threefifty_test)
rmse12 = rmse(knn12,threefifty_test)
rmse13 = rmse(knn13,threefifty_test)
rmse14 = rmse(knn14,threefifty_test)
rmse15 = rmse(knn15,threefifty_test)
rmse16 = rmse(knn16,threefifty_test)
rmse17 = rmse(knn17,threefifty_test)
rmse18 = rmse(knn18,threefifty_test)
rmse19 = rmse(knn19,threefifty_test)
rmse20 = rmse(knn20,threefifty_test)
rmse21 = rmse(knn21,threefifty_test)
rmse22 = rmse(knn22,threefifty_test)
rmse23 = rmse(knn23,threefifty_test)
rmse24 = rmse(knn24,threefifty_test)
rmse25 = rmse(knn25,threefifty_test)
```
\
Plotting RMSE versus K
```{r, echo=TRUE}
k <- c(2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25)
rmse <- c(rmse2,rmse3,rmse4,rmse5,rmse6,rmse7,rmse8,rmse9,rmse10,rmse11,rmse12,rmse13,rmse14,rmse15,rmse16,rmse17,rmse18,rmse19,rmse20,rmse21,rmse22,rmse23,rmse24,rmse25)
errors <- data.frame(k,rmse)
errors
ggplot(data=errors)+geom_line(aes(k,rmse))
```
\

It looks like the optimal value of K is 19.
\

## 65 AMG
Splitting the data into a training and a testing set
```{r, echo=TRUE}
ggplot(data=sixtyfive)+geom_point(mapping=aes(x=mileage,y=price))
sixtyfive_split = initial_split(sixtyfive,prop=0.9)
sixtyfive_train = training(sixtyfive_split)
sixtyfive_test = testing(sixtyfive_split)
```
\
Running K-nearest-neighbors, from K=2 to K=100
```{r, echo=TRUE}
knn2 = knnreg(price~mileage,data=sixtyfive_train,k=2)
knn3 = knnreg(price~mileage,data=sixtyfive_train,k=3)
knn4 = knnreg(price~mileage,data=sixtyfive_train,k=4)
knn5 = knnreg(price~mileage,data=sixtyfive_train,k=5)
knn6 = knnreg(price~mileage,data=sixtyfive_train,k=6)
knn7 = knnreg(price~mileage,data=sixtyfive_train,k=7)
knn8 = knnreg(price~mileage,data=sixtyfive_train,k=8)
knn9 = knnreg(price~mileage,data=sixtyfive_train,k=9)
knn10 = knnreg(price~mileage,data=sixtyfive_train,k=10)
knn11 = knnreg(price~mileage,data=sixtyfive_train,k=11)
knn12 = knnreg(price~mileage,data=sixtyfive_train,k=12)
knn13 = knnreg(price~mileage,data=sixtyfive_train,k=13)
knn14 = knnreg(price~mileage,data=sixtyfive_train,k=14)
knn15 = knnreg(price~mileage,data=sixtyfive_train,k=15)
knn16 = knnreg(price~mileage,data=sixtyfive_train,k=16)
knn17 = knnreg(price~mileage,data=sixtyfive_train,k=17)
knn18 = knnreg(price~mileage,data=sixtyfive_train,k=18)
knn19 = knnreg(price~mileage,data=sixtyfive_train,k=19)
knn20 = knnreg(price~mileage,data=sixtyfive_train,k=20)
knn21 = knnreg(price~mileage,data=sixtyfive_train,k=21)
knn22 = knnreg(price~mileage,data=sixtyfive_train,k=22)
knn23 = knnreg(price~mileage,data=sixtyfive_train,k=23)
knn24 = knnreg(price~mileage,data=sixtyfive_train,k=24)
knn25 = knnreg(price~mileage,data=sixtyfive_train,k=25)
```
\
Calculating the out-of-sample root mean-squared error (RMSE) for each value of k
```{r, echo=TRUE}
rmse2 = rmse(knn2,sixtyfive_test)
rmse3 = rmse(knn3,sixtyfive_test)
rmse4 = rmse(knn4,sixtyfive_test)
rmse5 = rmse(knn5,sixtyfive_test)
rmse6 = rmse(knn6,sixtyfive_test)
rmse7 = rmse(knn7,sixtyfive_test)
rmse8 = rmse(knn8,sixtyfive_test)
rmse9 = rmse(knn9,sixtyfive_test)
rmse10 = rmse(knn10,sixtyfive_test)
rmse11 = rmse(knn11,sixtyfive_test)
rmse12 = rmse(knn12,sixtyfive_test)
rmse13 = rmse(knn13,sixtyfive_test)
rmse14 = rmse(knn14,sixtyfive_test)
rmse15 = rmse(knn15,sixtyfive_test)
rmse16 = rmse(knn16,sixtyfive_test)
rmse17 = rmse(knn17,sixtyfive_test)
rmse18 = rmse(knn18,sixtyfive_test)
rmse19 = rmse(knn19,sixtyfive_test)
rmse20 = rmse(knn20,sixtyfive_test)
rmse21 = rmse(knn21,sixtyfive_test)
rmse22 = rmse(knn22,sixtyfive_test)
rmse23 = rmse(knn23,sixtyfive_test)
rmse24 = rmse(knn24,sixtyfive_test)
rmse25 = rmse(knn25,sixtyfive_test)
```
\
Plotting RMSE versus K
```{r, echo=TRUE}
k <- c(2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25)
rmse <- c(rmse2,rmse3,rmse4,rmse5,rmse6,rmse7,rmse8,rmse9,rmse10,rmse11,rmse12,rmse13,rmse14,rmse15,rmse16,rmse17,rmse18,rmse19,rmse20,rmse21,rmse22,rmse23,rmse24,rmse25)
errors <- data.frame(k,rmse)
errors
ggplot(data=errors)+geom_line(aes(k,rmse))
```
\
It looks like the optimal value of K is 12.