ETL : Make tidy data set
================
Akim van Eersel
September 5, 2020

*Note : the present markdown is a **.Rmd** file written in Rstudio, then
rendered by Knit in a *.md\* file, suited for Github visibility, and
saved along the twin R markdown file.\*

# Purpose overview

This readme file have the purpose to describe the methodology and the
code steps of **run\_analysis.R** file which return in output a tidy
data set formed in the text file **avgMeasures.txt**.  
To get more infos on the variables output of **avgMeasures.txt**, see
**variables\_codebook.txt**.  
This work is made for the peer-graded assignment course project of the
4th week lesson from the *Getting and Cleaning Data* course by John
Hopkins University on coursera plateform.  
The **run\_analysis.R** code purpose is to extract and transform data
from *Human Activity Recognition Using Smartphones Data Set Version 1.0*
research
[link](https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones),
source :

Jorge L. Reyes-Ortiz, Davide Anguita, Alessandro Ghio, Luca Oneto.  
Smartlab - Non Linear Complex Systems Laboratory  
DITEN - Università degli Studi di Genova.  
Via Opera Pia 11A, I-16145, Genoa, Italy.  
<activityrecognition@smartlab.ws>  
www.smartlab.ws

# Summary informations before running the code

## Data

Data folder was download manually as a zip file from the source. Then,
it was unzipped manually and directly move in the working directory
without any change.

## Snapshots of data frames

In order to show snapshot infos of objects in the running environment
will use the above code line. It’s only purpose is to give data frame
infos in a relatively compact form suited for the present markdwon file.
Theses code lines aren’t present in **run\_analysis.R** and only the
output is showed here for better readability.  
The output is a list with 3 explicit attributes which are :

1.  name = the data frame name showed
2.  dimensions = the dimensions of the data frame
3.  head *or* head\_truncated.columns = the first 2 lines of the data
    frame *or* the first 2 lines and 5 columns of big data frames

<!-- end list -->

``` r
list(name = dataframe_show_explicit_name, 
     dimensions = dim(dataframe),
     head = dataframe[1:2,]) ## or head_truncated.columns = dataframe[1:2,1:5])
```

## Running session

### System and IDE info

``` r
# System info
version
```

    ##                _                           
    ## platform       x86_64-w64-mingw32          
    ## arch           x86_64                      
    ## os             mingw32                     
    ## system         x86_64, mingw32             
    ## status                                     
    ## major          4                           
    ## minor          0.2                         
    ## year           2020                        
    ## month          06                          
    ## day            22                          
    ## svn rev        78730                       
    ## language       R                           
    ## version.string R version 4.0.2 (2020-06-22)
    ## nickname       Taking Off Again

``` r
# IDE info :
rstudioapi::versionInfo()$version
```

    ## [1] '1.3.1073'

### Files and folders info

``` r
# Working directory :
getwd()
```

    ## [1] "C:/Users/Akim Van Eersel/R_Projects/Data_Science_Specialization-John_Hopkins/Getting and Cleaning Data/Learning_ETL_EDA"

``` r
# Files in working directory :
list.files(getwd())
```

    ## [1] "avgMeasures.txt"        "Learning_ETL_EDA.Rproj" "README.md"             
    ## [4] "README.Rmd"             "run_analysis.R"         "UCI HAR Dataset"       
    ## [7] "variables_codebook.txt"

``` r
# Files in the unziped data source :
list.files("./UCI HAR Dataset")
```

    ## [1] "activity_labels.txt" "features.txt"        "features_info.txt"  
    ## [4] "README.txt"          "test"                "train"

# Code steps

## Libraries loaded

The following packages are loaded :

``` r
library(data.table)                 ## Mostly used to read txt with fread() faster than read.table(), but could be replaced easily
library(dplyr, warn.conflicts = F)  ## Mostly for merging tables, could be replace with a few more code
library(tidyr)                      ## To make tidy a messy data set
library(readr)                      ## To parse the numeric from a string, could be easily replace
```

## Reading data

All the data is read from **UCI HAR Dataset** folder and store :

``` r
datafolder <- "UCI HAR Dataset/"

trainfeature <- fread(file = paste(datafolder, "train/X_train.txt", sep = ""))
trainlabel <- fread(file = paste(datafolder, "train/y_train.txt", sep = ""))
trainsubject <- fread(file = paste(datafolder, "train/subject_train.txt", sep = ""))

testfeature <- fread(file = paste(datafolder, "test/X_test.txt", sep = ""))
testlabel <- fread(file = paste(datafolder, "test/y_test.txt", sep = ""))
testsubject <- fread(file = paste(datafolder, "test/subject_test.txt", sep = ""))

featurenames <- fread(file = paste(datafolder, "features.txt", sep = ""))
labelnames <- fread(file = paste(datafolder, "activity_labels.txt", sep = ""))
```

Quick overview on the loaded data sets :

    ## $name
    ## [1] "Features data set from training"
    ## 
    ## $dimensions
    ## [1] 7352  561
    ## 
    ## $head_truncated.columns
    ##           V1          V2         V3         V4         V5
    ## 1: 0.2885845 -0.02029417 -0.1329051 -0.9952786 -0.9831106
    ## 2: 0.2784188 -0.01641057 -0.1235202 -0.9982453 -0.9753002

    ## $name
    ## [1] "Features labels data set from training"
    ## 
    ## $dimensions
    ## [1] 7352    1
    ## 
    ## $head
    ##    V1
    ## 1:  5
    ## 2:  5

    ## $name
    ## [1] "Subject ids data set from training"
    ## 
    ## $dimensions
    ## [1] 7352    1
    ## 
    ## $head
    ##    V1
    ## 1:  1
    ## 2:  1

    ## $name
    ## [1] "Features data set from testing"
    ## 
    ## $dimensions
    ## [1] 2947  561
    ## 
    ## $head_truncated.columns
    ##           V1          V2          V3         V4         V5
    ## 1: 0.2571778 -0.02328523 -0.01465376 -0.9384040 -0.9200908
    ## 2: 0.2860267 -0.01316336 -0.11908252 -0.9754147 -0.9674579

    ## $name
    ## [1] "Features labels data set from testing"
    ## 
    ## $dimensions
    ## [1] 2947    1
    ## 
    ## $head
    ##    V1
    ## 1:  5
    ## 2:  5

    ## $name
    ## [1] "Subject ids data set from testing"
    ## 
    ## $dimensions
    ## [1] 2947    1
    ## 
    ## $head
    ##    V1
    ## 1:  2
    ## 2:  2

    ## $name
    ## [1] "Feature numeric labels and names"
    ## 
    ## $dimensions
    ## [1] 561   2
    ## 
    ## $head
    ##    V1                V2
    ## 1:  1 tBodyAcc-mean()-X
    ## 2:  2 tBodyAcc-mean()-Y

    ## $name
    ## [1] "Activity numeric labels and names"
    ## 
    ## $dimensions
    ## [1] 6 2
    ## 
    ## $head
    ##    V1               V2
    ## 1:  1          WALKING
    ## 2:  2 WALKING_UPSTAIRS

## Construct workable data and store in a single tidy data frame

### Combine train and test datasets (measurements, activity labels and subject ids)

``` r
combfeature <- bind_rows(trainfeature, testfeature)
comblabels <- bind_rows(trainlabel, testlabel)
combsubject <- bind_rows(trainsubject, testsubject)
```

Quick overview on the merged data sets :

    ## $name
    ## [1] "Merged features data sets"
    ## 
    ## $dimensions
    ## [1] 10299   561
    ## 
    ## $head_truncated.columns
    ##           V1          V2         V3         V4         V5
    ## 1: 0.2885845 -0.02029417 -0.1329051 -0.9952786 -0.9831106
    ## 2: 0.2784188 -0.01641057 -0.1235202 -0.9982453 -0.9753002

    ## $name
    ## [1] "Merged features labels data sets"
    ## 
    ## $dimensions
    ## [1] 10299     1
    ## 
    ## $head
    ##    V1
    ## 1:  5
    ## 2:  5

    ## $name
    ## [1] "Merged subject ids data sets"
    ## 
    ## $dimensions
    ## [1] 10299     1
    ## 
    ## $head
    ##    V1
    ## 1:  1
    ## 2:  1

### Set appropriate names for all datasets

``` r
## Rename activity label and names in concerned datasets : "V1", "V2" => "label", "activity"
colnames(labelnames) <- c("label", "activity")
colnames(comblabels) <- "label"

# Rename subject ids column : "V1" => "subjectid"
colnames(combsubject) <- "subjectid"

# Rename columns of the features files : "V1", "V2" => "measurement", "measurementname"
colnames(featurenames) <- c("measurement", "measurementname")
```

Quick overview on the variables renamed data sets :

    ## $name
    ## [1] "Renamed features labels data set"
    ## 
    ## $dimensions
    ## [1] 10299     1
    ## 
    ## $head
    ##    label
    ## 1:     5
    ## 2:     5

    ## $name
    ## [1] "Renamed subject ids data set"
    ## 
    ## $dimensions
    ## [1] 10299     1
    ## 
    ## $head
    ##    subjectid
    ## 1:         1
    ## 2:         1

    ## $name
    ## [1] "Renamed feature numeric labels and names"
    ## 
    ## $dimensions
    ## [1] 561   2
    ## 
    ## $head
    ##    measurement   measurementname
    ## 1:           1 tBodyAcc-mean()-X
    ## 2:           2 tBodyAcc-mean()-Y

    ## $name
    ## [1] "Renamed activity numeric labels and names"
    ## 
    ## $dimensions
    ## [1] 6 2
    ## 
    ## $head
    ##    label         activity
    ## 1:     1          WALKING
    ## 2:     2 WALKING_UPSTAIRS

### Merge by “label” the merged measurements data with the activity names

### Slice and store in different column the features/measurements names

The goal here is to split the features names in order get only one
variable by column.  
Example : “tBodyAcc-mean()-X” =\> “tBodyAcc”, “mean()”, “X”  
The first code line is a function splitting strings at each “-”
character.  
The second code line apply the splitting function for the feature names
column, get the appropriate sliced part by anonymous function and stored
it in a new column with corresponding name.

``` r
# Creation of a function to avoid repetition
cutstrcol <- function(coltosplit){strsplit(coltosplit, "-")}

featurenames <- featurenames %>%
      mutate(feature = unlist(lapply(cutstrcol(measurementname), function(elt){elt[1]})), 
             estimate = unlist(lapply(cutstrcol(measurementname), function(elt){elt[2]})), 
             estimateparameter = unlist(lapply(cutstrcol(measurementname), function(elt){elt[3]})))

# Delete feature full name column
featurenames[,"measurementname"] <- NULL
```

Quick overview on the updated data set :

    ## $name
    ## [1] "Sliced features names data set"
    ## 
    ## $dimensions
    ## [1] 561   4
    ## 
    ## $head
    ##    measurement  feature estimate estimateparameter
    ## 1:           1 tBodyAcc   mean()                 X
    ## 2:           2 tBodyAcc   mean()                 Y

### Combine subject ids, activity names and the measurements data

``` r
fulldata <- cbind(combsubject, comblabels[,2], combfeature)
```

Quick overview on the updated data set :

    ## $name
    ## [1] "Merged data set"
    ## 
    ## $dimensions
    ## [1] 10299   563
    ## 
    ## $head_truncated.columns
    ##    subjectid activity        V1          V2         V3
    ## 1:         1 STANDING 0.2885845 -0.02029417 -0.1329051
    ## 2:         1 STANDING 0.2784188 -0.01641057 -0.1235202

### Make tidy data where each parsed features are split and set as a unique variable with corresponding value

The first code line make a tidy data with all the features labels (still
are “V1” to “V561”) and the measurements values.  
The second code line parse the features labels to set only the numeric
piece as the name : “V1” =\> “1”, “V2” =\> “2”, etc …

``` r
fulldata <- fulldata %>% gather(key = measurement, value = value, -(subjectid:activity))
fulldata$measurement <- parse_number(fulldata$measurement)
```

Quick overview on the tidy data set :

    ## $name
    ## [1] "Tidy data set"
    ## 
    ## $dimensions
    ## [1] 5777739       4
    ## 
    ## $head
    ##   subjectid activity measurement     value
    ## 1         1 STANDING           1 0.2885845
    ## 2         1 STANDING           1 0.2784188

The next code block show the join of measurements tidy data set with
complete and parsed features names.

Quick overview on the merged tidy data set :

    ## $name
    ## [1] "Merged tidy data set"
    ## 
    ## $dimensions
    ## [1] 5777739       6
    ## 
    ## $head
    ##   subjectid activity  feature estimate estimateparameter     value
    ## 1         1 STANDING tBodyAcc   mean()                 X 0.2885845
    ## 2         1 STANDING tBodyAcc   mean()                 X 0.2784188

Finally, all data was loaded and merged to get a unique data set.  
The next step is to follow the last project instructions :

  - extract only the measurements on the mean and standard deviation for
    each measurement,
  - create a second, independent tidy data set with the average of each
    variable for each activity and each subject.

## Subset some measurement features and summarize their average according to different variables

### Search from all features measurements the mean and standard deviation calculations

``` r
extractdata <- fulldata[!is.na(fulldata$estimate) & (fulldata$estimate == "mean()" | fulldata$estimate == "std()"), ]
```

Then, group by all variables :

``` r
estimategrp <- extractdata %>% group_by(feature, estimateparameter, estimate, activity, subjectid)
```

Quick overview on the subsetted and grouped tidy data set :

    ## $name
    ## [1] "mean() and std() grouped tidy data set"
    ## 
    ## $dimensions
    ## [1] 679734      6
    ## 
    ## $head
    ## # A tibble: 2 x 6
    ## # Groups:   feature, estimateparameter, estimate, activity, subjectid [1]
    ##   subjectid activity feature  estimate estimateparameter value
    ##       <int> <chr>    <chr>    <chr>    <chr>             <dbl>
    ## 1         1 STANDING tBodyAcc mean()   X                 0.289
    ## 2         1 STANDING tBodyAcc mean()   X                 0.278

Finally, summarize the average value of each feature for either each
triaxial measurement or none triaxial specification, each activity and
each subject id.  
And write the data in a text file.

Even, if the output data can be seen in **avgMeasures.txt** text file,
here’s a quick overview :

    ## $name
    ## [1] "Summarize mean() and std() grouped tidy data set"
    ## 
    ## $dimensions
    ## [1] 11880     6
    ## 
    ## $head
    ## # A tibble: 2 x 6
    ## # Groups:   feature, estimateparameter, estimate, activity [1]
    ##   feature  estimateparameter estimate activity subjectid `mean(value)`
    ##   <chr>    <chr>             <chr>    <chr>        <int>         <dbl>
    ## 1 fBodyAcc X                 mean()   LAYING           1        -0.939
    ## 2 fBodyAcc X                 mean()   LAYING           2        -0.977
