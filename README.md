---
title: "Salary data analysis"
author: "Jean Cheptumo"
date: "`r Sys.Date()`"
output: html_document
---
```{r}
#Load the data
set.seed(12345)

library(dplyr)
library(tidyr)

employee_data <- read.csv("Salary_Data.csv")

#A view of number of rows,columns data types, column names and data
glimpse(employee_data) 

#A view of the first few rows of data
head(employee_data)
```
##Data Cleaning

```{r}
#Check for missing values by summing missing values in each column
colSums(is.na(employee_data))

#Find rows with missing values
missing <- employee_data[!complete.cases(employee_data),]
print(missing)

#Drop all rows with missing values
employee_data_1<- na.omit(employee_data)
glimpse(employee_data_1)

#Confirm there are no missing values
colSums(is.na(employee_data_1))
```

```{r}
#Confirm the data types are correct
sapply(employee_data_1,class)

#Convert Gender, Education and Job title to factor
employee_data_1[c("Gender","Education.Level","Job.Title")]<-
  lapply(employee_data_1[c("Gender","Education.Level","Job.Title")],as.factor)

#Convert Age,Years of experience and Salary to integer
employee_data_1[c("Age","Years.of.Experience","Salary")] <-
  lapply(employee_data_1[c("Age","Years.of.Experience","Salary")],as.integer)

```

```{r}
#Check, list and remove duplicates
duplicates <- employee_data_1[duplicated(employee_data_1) | duplicated(employee_data_1,fromLast = TRUE),]


#We choose not to drop the duplicates and consider that employees data may be similar.
```

```{r}
#Here, we learn that several rows were duplicates.
#This is the unique dataset of 1,788 rows

employee_data_2 <- employee_data_1 %>% distinct()


```

```{r}
#Check and handle outliers
#Visualize the Salary and Age distribution

boxplot(employee_data_1$Salary, main = "Salary distribution", ylab="Salary") #No outliers noted
boxplot(employee_data_1$Age, main= "Age distribution", ylab="Age") #Presence of outliers

#Calculate the IQR and the lower and upper bounds for the outliers
Q1 <- quantile(employee_data_1$Age,0.25)
Q3 <- quantile(employee_data_1$Age,0.75)
IQR= Q3-Q1

lower_bound <- Q1-1.5*IQR
upper_bound <- Q3 + 1.5*IQR

#Identification of the outliers
age_outliers<- employee_data_1$Age[employee_data_1$Age < lower_bound | employee_data_1$Age > upper_bound]

#Total count
outlier <- employee_data_1$Age < lower_bound |employee_data_1$Age > upper_bound
sum(outlier) #123 ages are outlier

#Drop the outliers under the Age column
employee_data_no_outlier <-
  employee_data_1[employee_data_1$Age >= lower_bound & employee_data_1$Age <= upper_bound,]

head(employee_data_no_outlier)

employee_data_clean <- employee_data_1 %>%
  filter(Age >= lower_bound & Age <= upper_bound)

boxplot(employee_data_clean$Age, main="Age distribution", ylab="Age")
```

```{r}
#Clean the education level names
summary(employee_data_clean$Education.Level)


library(forcats)

employee_data_clean$Education.Level<-
  fct_recode(employee_data_clean$Education.Level,
             "Bachelor's"="Bachelor's Degree",
             "Master's"="Master's Degree",
             "PhD" = "phD")
#Drop the rows where education.level is blank

new_employee_data <- employee_data_clean %>%
  filter(Education.Level != "")

summary(new_employee_data)
```
##Exploratory Data Analysis
```{r}
# Descriptive analysis
## Summary statistics for the integer columns
summary(new_employee_data)

## Visualize the distribution of key variables
hist(new_employee_data$Salary, main="Salary distribution", xlab="Salary", col="lightblue", breaks=10)

hist(new_employee_data$Age,main="Age distribution", xlab="Age", col="lightgreen", breaks = 10)

```
## Visualize the relationship between variables 

```{r}
#Salary versus years of experience
plot(new_employee_data$Years.of.Experience,new_employee_data$Salary, main="Salary vs Experience", xlab="Experience", ylab="Salary")

boxplot(Salary~Gender, data= new_employee_data, main="Salary by Gender", xlab= "Gender", ylab="Salary")

boxplot(Salary ~ Education.Level, data =new_employee_data, main="Salary by Education level",xlab="Education level",ylab="Salary")
```

## Correlation Analysis
```{r}
# Look at the correlations between numeric variables(Salary,Age,Years of experience)
## Correlation matrix (correlation coefficient) to measure the strength & direction of the linear relationship
cor(new_employee_data[,c("Age","Salary","Years.of.Experience")])

```
#Statistical Analysis
```{r}
#Linear regression
#Predict the Salary based on Years of experience and Education level
model <- lm(Salary ~ Years.of.Experience + Education.Level, data = new_employee_data)
summary(model)

```

#ANOVA for categorical variales
```{r}
#Test the effect of categorical variables on Salary

anova <- aov(Salary ~ Education.Level, data = new_employee_data)

summary(anova)

anova_1 <-aov(Salary ~ Education.Level + Gender, data= new_employee_data)

summary(anova_1)

```
#Export the final dataset
```{r}
write.csv(new_employee_data,"Employee Data_clean.csv", row.names = FALSE)
```
