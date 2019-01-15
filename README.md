# airBnb-and-Zillow-data-challenge
You are consulting for a real estates agency and you have to predict which zipcodes are the best profitable in investing. You are given 2 datasets for this.

MITA Capstone Project - Air Bnb and zillow data challenge
=========================================================

Submitted by - Dhiraj Tripathi
------------------------------

Ruid - 182002411
----------------

Under supervision of Professor Sergei Schreider
-----------------------------------------------

The data challenge is about a real estate agency willing to invest in
New York city. The agency has already learnt that investing in 2 Bedroom
properties within New York is profitable. This allows us to narrow our
search and focus on other important factors. Our main agenda is to find
out the zipcodes which are the most profitable in terms of investment.

We are given two datasets for this challenge:

1.  Airbnb :- This dataset contains information about property listings
    along with other variables like size rank, location, cleaning fee,
    rental price on daily, weekly and monthly basis, reviews and
    ratings, zipcodes etc.

2.  Zillow data: This dataset contains full price information to own a
    property in many different zipcodes. It contains the historical
    price data of the properties by zipcode from April 1996 to
    June 2017. Each column represents the price of the properties lised
    in different zipcode for a particular month.

To achieve this, we have to clean the data first, analyze it and
visualize it later to get some insights on the zipcodes.

Lets get started with this data challenge by by loading and installing
all of the required packages as below:

    if(!require("plyr")){ # for data analysis
      
      install.packages("tidyverse",dependencies = T)
      
      library(plyr)
      
    }

    ## Loading required package: plyr

    detach(package:plyr) #remove it because it yields issues with tidyverse; however, still need to make sure it's installed for rbind.fill

    if(!require("colorspace")){ # is required in system to install another dependent package tidyverse
      
      install.packages("colorspace",dependencies = T)
      
      library(colorspace)
      
    }

    ## Loading required package: colorspace

    if(!require("tidyverse")){
      # for data manipulation and aggregation
      install.packages("tidyverse",dependencies = T)
      
      library(tidyverse)
      
    }

    ## Loading required package: tidyverse

    ## -- Attaching packages ---------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 3.0.0     v purrr   0.2.5
    ## v tibble  1.4.2     v dplyr   0.7.6
    ## v tidyr   0.8.1     v stringr 1.3.1
    ## v readr   1.1.1     v forcats 0.3.0

    ## -- Conflicts ------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    if(!require("plotly")){
      # for interactive graphics
      install.packages("plotly",dependencies = T)
      
      library(plotly)
      
    }

    ## Loading required package: plotly

    ## 
    ## Attaching package: 'plotly'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     last_plot

    ## The following object is masked from 'package:stats':
    ## 
    ##     filter

    ## The following object is masked from 'package:graphics':
    ## 
    ##     layout

    if(!require("forecast")){
      # for time series analysis
      install.packages("forecast",dependencies = T)
      
      library(forecast)
    }

    ## Loading required package: forecast

    if(!require("astsa")){
      # for time series analysis
      install.packages("astsa",dependencies = T)
      
      library(astsa)
    }

    ## Loading required package: astsa

    ## 
    ## Attaching package: 'astsa'

    ## The following object is masked from 'package:forecast':
    ## 
    ##     gas

    if(!require("data.table")){
      # to read large data
      install.packages("data.table",dependencies = T)
      
      library(data.table)
      
    }

    ## Loading required package: data.table

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

    if(!require("Amelia")){
      # for imputing missing values
      install.packages("Amelia",dependencies = T)
      
      library(Amelia)
      
    }

    ## Loading required package: Amelia

    ## Loading required package: Rcpp

    ## ## 
    ## ## Amelia II: Multiple Imputation
    ## ## (Version 1.7.5, built: 2018-05-07)
    ## ## Copyright (C) 2005-2019 James Honaker, Gary King and Matthew Blackwell
    ## ## Refer to http://gking.harvard.edu/amelia/ for more information
    ## ##

    if(!require("mice")){
      # for imputing missing values
      install.packages("mice",dependencies = T)
      
      library(mice)
      
    }

    ## Loading required package: mice

    ## Loading required package: lattice

    ## 
    ## Attaching package: 'mice'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     complete

    ## The following objects are masked from 'package:base':
    ## 
    ##     cbind, rbind

    if(!require("dplyr")){
      # data aggregation
      install.packages("dplyr",dependencies = T)
      
      library(dplyr)
      
    }

    install.packages("webshot")

    ## Installing package into 'C:/Users/dhira/OneDrive/Documents/R/win-library/3.5'
    ## (as 'lib' is unspecified)

    ## package 'webshot' successfully unpacked and MD5 sums checked
    ## 
    ## The downloaded binary packages are in
    ##  C:\Users\dhira\AppData\Local\Temp\Rtmpes6F8s\downloaded_packages

    webshot::install_phantomjs()

    ## phantomjs has been installed to C:\Users\dhira\AppData\Roaming\PhantomJS

    getwd()

    ## [1] "F:/CAP1/Final Submission"

Before we delve further into analysis, lets list out the assumptions
given by CAP1 and also the assumptions that I made.

Assumptions:
============

1.  The investor will pay for the property in cash.  
2.  $1 today is worth the same 100 years from now.  
3.  All properties and all square feet within each locale can be assumed
    to be homogeneous.  
4.  Occupancy rate is 75%  
5.  We assume that a property gets rented 40% of the times on a daily
    basis, 40% of the times on a weekly basis and 20% of the times on a
    monthly basis.  
6.  Size Rank is not given in Zillow data hence not considered as a
    factor in revenue estimation.

Reading the CSV files
---------------------

    listData<-read.csv("F:/CAP1/listings.csv")
    zillData <- read.csv("F:/CAP1/airbnb-zillow-data-challenge-master/Zip_Zhvi_2bedroom.csv")

### Check the dimensions of the data:

    dim (listData)

    ## [1] 40753    95

    dim (zillData)

    ## [1] 8946  262

As we see , there are too many columns in both the data sets. We will
only select the ones which are relevant to us.

We will first work on the Zillow data and extract the columns. Meaning,
we will only select the price data for the 3 years and based on that we
will forecast the price of the property in 2018.

    funct1<-function(temporarydf){
      
      {
      
      temporarydf<-temporarydf[,c(2,3,7,226:262)]

      temporarydf<-subset(temporarydf,City=="New York")

     }

      return(temporarydf)
    }

    ABC<-funct1(zillData) ## Call the function on Zillow Data
    head(ABC,5)

    ##    RegionName     City SizeRank X2014.06 X2014.07 X2014.08 X2014.09
    ## 1       10025 New York        1  1135100  1130000  1138200  1153700
    ## 3       10023 New York        3  1745900  1753800  1736600  1730400
    ## 14      10128 New York       14  1221200  1230500  1243500  1259000
    ## 15      10011 New York       15  2043500  2056300  2064500  2066000
    ## 21      10003 New York       21  1772200  1762700  1736700  1712400
    ##    X2014.10 X2014.11 X2014.12 X2015.01 X2015.02 X2015.03 X2015.04 X2015.05
    ## 1   1174800  1185400  1188400  1189700  1193700  1199900  1201400  1202600
    ## 3   1734500  1728700  1720800  1717700  1700100  1680400  1676400  1685600
    ## 14  1277400  1296300  1305600  1310800  1313400  1313500  1314500  1328000
    ## 15  2057900  2031300  1999000  1979200  1982900  2001600  2014700  2023500
    ## 21  1703700  1702500  1708800  1716300  1720500  1721800  1741800  1775800
    ##    X2015.06 X2015.07 X2015.08 X2015.09 X2015.10 X2015.11 X2015.12 X2016.01
    ## 1   1214200  1235200  1258000  1287700  1307200  1313900  1317100  1327400
    ## 3   1708100  1730400  1751800  1778300  1810400  1831600  1844400  1861600
    ## 14  1347900  1376100  1409500  1431400  1441600  1453100  1468100  1492000
    ## 15  2055300  2078300  2083600  2088800  2110600  2127500  2168900  2204700
    ## 21  1796500  1821500  1870100  1901000  1904900  1914000  1926400  1932200
    ##    X2016.02 X2016.03 X2016.04 X2016.05 X2016.06 X2016.07 X2016.08 X2016.09
    ## 1   1338800  1350400  1356600  1358500  1364000  1373300  1382600  1374400
    ## 3   1889600  1901500  1895300  1890200  1898400  1924500  1967300  1993500
    ## 14  1518100  1531300  1525300  1509000  1520400  1543900  1547400  1526000
    ## 15  2216100  2212500  2222600  2231900  2250800  2285200  2329100  2354000
    ## 21  1936700  1945200  1935600  1911200  1918700  1947600  1951300  1932800
    ##    X2016.10 X2016.11 X2016.12 X2017.01 X2017.02 X2017.03 X2017.04 X2017.05
    ## 1   1364100  1366300  1354800  1327500  1317300  1333700  1352100  1390000
    ## 3   1980700  1960900  1951300  1937800  1929800  1955000  2022400  2095000
    ## 14  1523700  1527200  1541600  1557800  1582900  1598900  1646100  1720500
    ## 15  2355500  2352200  2332100  2313300  2319600  2342100  2365900  2419700
    ## 21  1930400  1937500  1935100  1915700  1916500  1965700  2045300  2109100
    ##    X2017.06
    ## 1   1431000
    ## 3   2142300
    ## 14  1787100
    ## 15  2480400
    ## 21  2147000

Now, we will use the ARIMA model on this extracted dataframe and predict
the price data 12 steps ahead and store it in a new dataframe. This new
dataframe will be used further in merging the datasets.

    for(i in 1:nrow(ABC)){
      
      {

    Q1<-ABC[,c(4:40)]

    W1<-ts(Q1[,c(1:37)],frequency = 12)

    ## We used the auto.arima function in a different script to find out them best arima model and hence the order (1,0,0)  in the arima model.

    ARIMA1<-arima((W1[,i]),order = c(1,0,0), seasonal = list(order=c(1,0,0),period=NA),method = "ML")

    pred = predict(ARIMA1,n.ahead = 12)
    predictvalue<-pred$pred

    ABC$EstimatedPrice[i] = predictvalue[length(predictvalue)]
      }
      
      newABC<-subset(ABC[,c(1,2,3,41)])
      colnames(newABC)[colnames(newABC)=="RegionName"]<-"zipcode"
    }

    head(newABC)

    ##    zipcode     City SizeRank EstimatedPrice
    ## 1    10025 New York        1        1439676
    ## 3    10023 New York        3        1449927
    ## 14   10128 New York       14        1450338
    ## 15   10011 New York       15        1454761
    ## 21   10003 New York       21        1463041
    ## 32   11201 New York       32        1465116

Now that we have the new data frame with forecasted values extracted
from the Zillow data, we will move with extracting the Airbnb data so
that we can merge both of them and analyze further.

    filterList <- function(tempdf){
      # Select only relevant columns
      relevantcol <- c("id","zipcode","bedrooms","square_feet","price","weekly_price","monthly_price","cleaning_fee","number_of_reviews","review_scores_rating")
      tempdf <- tempdf[,relevantcol]
      # filter data containing 2 bedrooms
      tempdf <- subset(tempdf,tempdf$bedrooms=="2")
      return(tempdf)
    }

    filteredListData <- filterList(listData) # call the function

    str(filteredListData) # observe the structure of this data

    ## 'data.frame':    4894 obs. of  10 variables:
    ##  $ id                  : int  9513511 5046189 4357134 16027061 11301089 14855080 16231738 2836845 1841252 1581579 ...
    ##  $ zipcode             : Factor w/ 205 levels "","10000","10001",..: 82 89 102 102 106 106 106 106 106 106 ...
    ##  $ bedrooms            : int  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ square_feet         : int  NA NA NA NA NA NA NA NA NA 1000 ...
    ##  $ price               : Factor w/ 583 levels "$1,000.00 ","$1,021.00 ",..: 65 87 151 205 516 178 33 98 33 207 ...
    ##  $ weekly_price        : Factor w/ 786 levels "","$1,000.00 ",..: 1 1 1 1 1 1 1 29 1 215 ...
    ##  $ monthly_price       : Factor w/ 839 levels "","$1,000.00 ",..: 1 1 1 1 1 1 1 492 1 724 ...
    ##  $ cleaning_fee        : Factor w/ 172 levels "","$0.00 ","$10.00 ",..: 1 148 1 1 108 168 144 148 64 159 ...
    ##  $ number_of_reviews   : int  4 31 0 0 1 15 2 107 3 135 ...
    ##  $ review_scores_rating: int  85 95 NA NA 60 87 80 96 90 97 ...

    head(filteredListData)

    ##          id zipcode bedrooms square_feet    price weekly_price
    ## 13  9513511   10462        2          NA $130.00              
    ## 24  5046189   10469        2          NA $150.00              
    ## 34  4357134   11102        2          NA $200.00              
    ## 37 16027061   11102        2          NA $250.00              
    ## 39 11301089   11105        2          NA  $79.00              
    ## 41 14855080   11105        2          NA $225.00              
    ##    monthly_price cleaning_fee number_of_reviews review_scores_rating
    ## 13                                            4                   85
    ## 24                    $75.00                 31                   95
    ## 34                                            0                   NA
    ## 37                                            0                   NA
    ## 39                   $400.00                  1                   60
    ## 41                    $95.00                 15                   87

Merge the Data
==============

    finalData<-merge(filteredListData,newABC,by=c("zipcode"))
    str(finalData) ## see the structure again

    ## 'data.frame':    1238 obs. of  13 variables:
    ##  $ zipcode             : Factor w/ 205 levels "","10000","10001",..: 6 6 6 6 6 6 6 6 6 6 ...
    ##  $ id                  : int  13561752 4942107 711635 4510857 3799598 568743 8335547 15094880 7664343 8884228 ...
    ##  $ bedrooms            : int  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ square_feet         : int  NA NA 800 NA NA NA NA NA NA NA ...
    ##  $ price               : Factor w/ 583 levels "$1,000.00 ","$1,021.00 ",..: 378 579 194 53 194 96 205 182 120 263 ...
    ##  $ weekly_price        : Factor w/ 786 levels "","$1,000.00 ",..: 1 1 110 716 1 147 1 1 2 164 ...
    ##  $ monthly_price       : Factor w/ 839 levels "","$1,000.00 ",..: 1 1 646 452 1 675 1 1 1 659 ...
    ##  $ cleaning_fee        : Factor w/ 172 levels "","$0.00 ","$10.00 ",..: 37 1 121 133 148 148 159 121 4 4 ...
    ##  $ number_of_reviews   : int  14 37 63 2 144 137 1 26 2 72 ...
    ##  $ review_scores_rating: int  94 100 95 80 88 83 100 95 90 95 ...
    ##  $ City                : Factor w/ 4684 levels "Aberdeen","Abilene",..: 2702 2702 2702 2702 2702 2702 2702 2702 2702 2702 ...
    ##  $ SizeRank            : int  21 21 21 21 21 21 21 21 21 21 ...
    ##  $ EstimatedPrice      : num  1463041 1463041 1463041 1463041 1463041 ...

    summary(finalData) ##print the summary to get the insights.

    ##     zipcode          id              bedrooms  square_feet    
    ##  11215  :141   Min.   :   20853   Min.   :2   Min.   :   0.0  
    ##  10003  :133   1st Qu.: 4218751   1st Qu.:2   1st Qu.: 800.0  
    ##  10025  :112   Median : 9410246   Median :2   Median : 975.0  
    ##  10036  :108   Mean   : 9218383   Mean   :2   Mean   : 921.6  
    ##  10011  :102   3rd Qu.:14476782   3rd Qu.:2   3rd Qu.:1100.0  
    ##  10014  : 95   Max.   :18508770   Max.   :2   Max.   :1600.0  
    ##  (Other):547                                  NA's   :1210    
    ##       price         weekly_price    monthly_price    cleaning_fee
    ##  $250.00 : 93             :981             :1058           :236  
    ##  $200.00 : 72   $1,200.00 : 14   $4,000.00 :  12   $100.00 :208  
    ##  $300.00 : 59   $1,500.00 : 13   $6,000.00 :   8   $50.00  : 87  
    ##  $150.00 : 56   $1,100.00 : 10   $3,000.00 :   6   $80.00  : 81  
    ##  $350.00 : 42   $1,000.00 :  9   $5,000.00 :   6   $150.00 : 80  
    ##  $400.00 : 32   $1,400.00 :  8   $2,500.00 :   5   $75.00  : 73  
    ##  (Other) :884   (Other)   :203   (Other)   : 143   (Other) :473  
    ##  number_of_reviews review_scores_rating       City         SizeRank     
    ##  Min.   :  0.00    Min.   : 20.00       New York:1238   Min.   :   1.0  
    ##  1st Qu.:  1.00    1st Qu.: 90.00       Aberdeen:   0   1st Qu.:  15.0  
    ##  Median :  5.00    Median : 95.00       Abilene :   0   Median :  71.0  
    ##  Mean   : 17.43    Mean   : 93.12       Abingdon:   0   Mean   : 485.9  
    ##  3rd Qu.: 20.00    3rd Qu.:100.00       Abington:   0   3rd Qu.: 580.0  
    ##  Max.   :306.00    Max.   :100.00       Acton   :   0   Max.   :4149.0  
    ##                    NA's   :268          (Other) :   0                   
    ##  EstimatedPrice   
    ##  Min.   :1439676  
    ##  1st Qu.:1454761  
    ##  Median :1463041  
    ##  Mean   :1484455  
    ##  3rd Qu.:1486928  
    ##  Max.   :1595637  
    ## 

We can notice that there are a lot of NA values in the finalData. So the
next step is to clean the data.

Data Cleaning
=============

    colnames(finalData) <-  c("zipcode","id","bedrooms","square_feet","price","weekly_price","monthly_price","cleaning_fee","number_of_reviews","review_scores_rating","city","size_rank","current_price")  ## change col names
    finalData$city <- factor(finalData$city, levels=c("New York")) ## set a filter where city is specifically New York

Select the columns with price information to eliminated the $ symbols.

    cols <- c("price", "weekly_price", "monthly_price","cleaning_fee")

    replace_dollar<-function(x){
      price<-as.numeric(gsub("[$,]","",x))
      return(price)
    }

    finalData[cols] <- lapply(finalData[cols], replace_dollar) ## call the function to replace the symbols with white spaces

    str(finalData) ## see the structure

    ## 'data.frame':    1238 obs. of  13 variables:
    ##  $ zipcode             : Factor w/ 205 levels "","10000","10001",..: 6 6 6 6 6 6 6 6 6 6 ...
    ##  $ id                  : int  13561752 4942107 711635 4510857 3799598 568743 8335547 15094880 7664343 8884228 ...
    ##  $ bedrooms            : int  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ square_feet         : int  NA NA 800 NA NA NA NA NA NA NA ...
    ##  $ price               : num  450 989 240 119 240 159 250 229 180 305 ...
    ##  $ weekly_price        : num  NA NA 1365 850 NA ...
    ##  $ monthly_price       : num  NA NA 5460 3200 NA ...
    ##  $ cleaning_fee        : num  150 NA 50 60 75 75 85 50 100 100 ...
    ##  $ number_of_reviews   : int  14 37 63 2 144 137 1 26 2 72 ...
    ##  $ review_scores_rating: int  94 100 95 80 88 83 100 95 90 95 ...
    ##  $ city                : Factor w/ 1 level "New York": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ size_rank           : int  21 21 21 21 21 21 21 21 21 21 ...
    ##  $ current_price       : num  1463041 1463041 1463041 1463041 1463041 ...

As we observe the structure of this data, we will realize that certain
columns like "number of reviews" need to be scaled as the numbers in
this column can affect the calculations.

    normalize <- function(x){
      return((x-min(x))/(max(x)-min(x))) # function to scale variables between 0 and 1
    }
    finalData["number_of_reviews"] <- lapply(finalData["number_of_reviews"], normalize) 

    summary(finalData$number_of_reviews)

    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## 0.000000 0.003268 0.016340 0.056952 0.065360 1.000000

This shows that scaling has been applied successfully.So we should move
forward with imputing the missing data.

    missingValues <- as.data.frame(colSums(sapply(finalData,is.na)))
    library(data.table)
    missingValues <- as.data.frame(setDT(missingValues, keep.rownames = TRUE)) ## set a data table with the rownames included
    colnames(missingValues)<-c("columnName","totalNA_values")##set the col names

    library(dplyr)

    install.packages("VIM",repos = "http://cran.us.r-project.org")

    ## Installing package into 'C:/Users/dhira/OneDrive/Documents/R/win-library/3.5'
    ## (as 'lib' is unspecified)

    ## package 'VIM' successfully unpacked and MD5 sums checked

    ## Warning: cannot remove prior installation of package 'VIM'

    ## 
    ## The downloaded binary packages are in
    ##  C:\Users\dhira\AppData\Local\Temp\Rtmpes6F8s\downloaded_packages

    library(VIM) ## lets you visualize the missing data. This is part of the package MICE

    ## Error in library(VIM): there is no package called 'VIM'

    mice_plot <- aggr(finalData, col=c('navyblue','yellow'),
                      numbers=TRUE, sortVars=TRUE,
                      cex.axis=.7,
                      gap=3)

    ## Error in aggr(finalData, col = c("navyblue", "yellow"), numbers = TRUE, : could not find function "aggr"

We now move forward with imputing the NA values.

    missingValues<-missingValues%>%
      ## mutate function lets you modify an existing column and we can also write a function to replace the NA values
      mutate_at(vars(totalNA_values),funs(percentNA_values=.*100/nrow(finalData)))%>%
      arrange(desc(percentNA_values))

    head(missingValues,13)

    ##              columnName totalNA_values percentNA_values
    ## 1           square_feet           1210         97.73829
    ## 2         monthly_price           1058         85.46042
    ## 3          weekly_price            981         79.24071
    ## 4  review_scores_rating            268         21.64782
    ## 5          cleaning_fee            236         19.06300
    ## 6               zipcode              0          0.00000
    ## 7                    id              0          0.00000
    ## 8              bedrooms              0          0.00000
    ## 9                 price              0          0.00000
    ## 10    number_of_reviews              0          0.00000
    ## 11                 city              0          0.00000
    ## 12            size_rank              0          0.00000
    ## 13        current_price              0          0.00000

    library(mice)

    dataSet<-subset(finalData, select = -c(id,city)) ##removing id and city columns so that we can impute the numeric values in the columns which have missing data

    imputeddataSet <- mice(dataSet, m=5, method='cart', printFlag=FALSE) ## use the CART method from MICE package to impute the missing values. See below for the number of logged events.

    ## Warning: Number of logged events: 136

    complete_dataSet<-complete(imputeddataSet) ## complete the imputed dataset
    finaldf_subset<-subset(finalData,select = c(id,city)) ##include the id and city

    finaldf_complete <- cbind(complete_dataSet,finaldf_subset) # combining the imputed dataset to add id and city

    sum(sapply(finaldf_complete, function(x) { sum(is.na(x)) })) # Check if there are any more NA values 

    ## [1] 0

    summary(finaldf_complete) # Check the summary of the dataframe again

    ##     zipcode       bedrooms  square_feet         price       
    ##  11215  :141   Min.   :2   Min.   :   0.0   Min.   :  28.0  
    ##  10003  :133   1st Qu.:2   1st Qu.: 900.0   1st Qu.: 165.0  
    ##  10025  :112   Median :2   Median :1000.0   Median : 240.0  
    ##  10036  :108   Mean   :2   Mean   : 988.5   Mean   : 278.7  
    ##  10011  :102   3rd Qu.:2   3rd Qu.:1150.0   3rd Qu.: 325.0  
    ##  10014  : 95   Max.   :2   Max.   :1600.0   Max.   :4700.0  
    ##  (Other):547                                                
    ##   weekly_price  monthly_price    cleaning_fee    number_of_reviews 
    ##  Min.   : 310   Min.   : 1250   Min.   :  0.00   Min.   :0.000000  
    ##  1st Qu.:1100   1st Qu.: 3490   1st Qu.: 60.00   1st Qu.:0.003268  
    ##  Median :1500   Median : 5000   Median : 90.00   Median :0.016340  
    ##  Mean   :1757   Mean   : 5680   Mean   : 93.19   Mean   :0.056952  
    ##  3rd Qu.:2200   3rd Qu.: 7000   3rd Qu.:110.00   3rd Qu.:0.065360  
    ##  Max.   :5950   Max.   :17100   Max.   :350.00   Max.   :1.000000  
    ##                                                                    
    ##  review_scores_rating   size_rank      current_price    
    ##  Min.   : 20.00       Min.   :   1.0   Min.   :1439676  
    ##  1st Qu.: 90.00       1st Qu.:  15.0   1st Qu.:1454761  
    ##  Median : 96.00       Median :  71.0   Median :1463041  
    ##  Mean   : 93.05       Mean   : 485.9   Mean   :1484455  
    ##  3rd Qu.:100.00       3rd Qu.: 580.0   3rd Qu.:1486928  
    ##  Max.   :100.00       Max.   :4149.0   Max.   :1595637  
    ##                                                         
    ##        id                 city     
    ##  Min.   :   20853   New York:1238  
    ##  1st Qu.: 4218751                  
    ##  Median : 9410246                  
    ##  Mean   : 9218383                  
    ##  3rd Qu.:14476782                  
    ##  Max.   :18508770                  
    ## 

Aggregating the data
====================

    library(dplyr)

    # mean of current price and other price attributes

    avg_df<-finaldf_complete%>%
            group_by(zipcode)%>%
            summarise_at(vars(square_feet:current_price),mean)

    # count number of properties in each zipcode as unique_id

    unique_id_df<-finaldf_complete%>% select(zipcode,id)%>%
              group_by(zipcode)%>%
              mutate(vars(id=n_distinct(id)))%>%
              select(zipcode,id)%>%
              distinct()
    summary_df<-inner_join(avg_df,unique_id_df,by="zipcode")


    summary(summary_df)

    ##     zipcode     square_feet         price        weekly_price 
    ##  11215  :141   Min.   : 882.1   Min.   : 65.0   Min.   : 350  
    ##  10003  :133   1st Qu.: 955.6   1st Qu.:209.5   1st Qu.:1341  
    ##  10025  :112   Median : 998.2   Median :286.0   Median :1862  
    ##  10036  :108   Mean   : 988.5   Mean   :278.7   Mean   :1757  
    ##  10011  :102   3rd Qu.:1027.6   3rd Qu.:356.1   3rd Qu.:2116  
    ##  10014  : 95   Max.   :1200.0   Max.   :367.3   Max.   :2286  
    ##  (Other):547                                                  
    ##  monthly_price   cleaning_fee    number_of_reviews review_scores_rating
    ##  Min.   :1250   Min.   : 25.00   Min.   :0.00000   Min.   : 80.00      
    ##  1st Qu.:4391   1st Qu.: 83.21   1st Qu.:0.04566   1st Qu.: 91.40      
    ##  Median :6144   Median : 95.25   Median :0.05497   Median : 93.38      
    ##  Mean   :5680   Mean   : 93.19   Mean   :0.05695   Mean   : 93.05      
    ##  3rd Qu.:6978   3rd Qu.:103.69   3rd Qu.:0.07383   3rd Qu.: 94.72      
    ##  Max.   :7230   Max.   :112.96   Max.   :0.23366   Max.   :100.00      
    ##                                                                        
    ##    size_rank      current_price           id          
    ##  Min.   :   1.0   Min.   :1439676   Min.   :   20853  
    ##  1st Qu.:  15.0   1st Qu.:1454761   1st Qu.: 4218751  
    ##  Median :  71.0   Median :1463041   Median : 9410246  
    ##  Mean   : 485.9   Mean   :1484455   Mean   : 9218383  
    ##  3rd Qu.: 580.0   3rd Qu.:1486928   3rd Qu.:14476782  
    ##  Max.   :4149.0   Max.   :1595637   Max.   :18508770  
    ## 

As we observe the summary\_df, we can notice that most of the zipcodes
in New York have more than 50 properties listed. Daily price ranges from
$65 to $367.3

Exploratory Data Analysis
=========================

We now define our new variables in order to construct a model which
estimates the revenue based on Airbnb data.

    ## Based on the assumptions as mentioned in the beginning

    p_daily=.40 
    p_weekly=.40
    p_monthly=.20
    occupancy_rate<-.75
    Quarter_1days<-90
    Quarter_2days<-180
    Year_days<-365

    summary_df$TotalCost<-summary_df$current_price*1.2 ## inflate the price to get the max cost possible to be on the safer side while calculating

    summary_df$Review_effect <- normalize(summary_df$review_scores_rating) # scale the review_scores_rating

    summary_df$Review_effect<-ifelse(summary_df$Review_effect>0,summary_df$Review_effect,mean(summary_df$Review_effect)) ## if the review effect is scaled to 0 , then substitute it with the mean 

    summary_df$Revenue_by_q1<-occupancy_rate*Quarter_1days*((p_daily*summary_df$price)+(p_weekly*summary_df$weekly_price/7)+(p_monthly*summary_df$monthly_price/30))*summary_df$Review_effect #calculate the fisrt quarter revenue

    summary_df$Revenue_by_q2<-occupancy_rate*Quarter_2days*((p_daily*summary_df$price)+(p_weekly*summary_df$weekly_price/7)+(p_monthly*summary_df$monthly_price/30))*summary_df$Review_effect #calculate the second quarter revenue

    summary_df$Revenue_by_year<-occupancy_rate*Year_days*((p_daily*summary_df$price)+(p_weekly*summary_df$weekly_price/7)+(p_monthly*summary_df$monthly_price/30))*summary_df$Review_effect #calculate the yearly revenue

    # Obtain the Revenue by amount spend ratio for first quarter

    summary_df$Revenue_by_Cost_RatioQ1<-normalize(summary_df$Revenue_by_q1/summary_df$TotalCost)    

    # percentage of properties listed for the given zipcode 

    summary_df$Percent_units <- normalize(summary_df$id*100/sum(summary_df$id))

    summary_df$Cost_by_Revenue <- summary_df$TotalCost/summary_df$Revenue_by_year

We have constructed the model to determine revenue and other paramters.
We will thus move on the Visualization part.

Visualizing the data
====================

    plot_my_graph <- function(col_name){
      # sort the dataframe for by col_name in descending order and subset for top N zipcodes
      
      v <- enquo(col_name)
      
      n=10 # select top n values
      
      df_sorted_unique_id <- arrange(summary_df[summary_df$id>10,],desc(!!v)) [1:n,] # order data
      
      # reassign factor levels
      
      df_sorted_unique_id$zipcode <- factor(df_sorted_unique_id$zipcode)
      
      
      # Return zipcodes
      
      return(df_sorted_unique_id)
      
    }

We write a function to order any relevant column and return zipcodes and
sort the listings. This ordering function will be used to visualize the
data further.

    library(plotly)

Visualize the trend between Revenue and Total Cost
==================================================

    plot_ly(summary_df, y = ~Revenue_by_q1, x = ~TotalCost, text = ~zipcode, type = 'scatter', mode = 'markers', size = 2000,
            color = ~Revenue_by_Cost_RatioQ1, 
            marker = list(opacity = 0.4, sizemode = 'diameter')) %>%
      layout(title = 'Revenue vs Total Cost',
             xaxis = list(showgrid = T),
             yaxis = list(showgrid = T),
             showlegend = T)

![code chunk-1](https://user-images.githubusercontent.com/39188654/51153175-25565f80-183d-11e9-80f9-0d238aedb6bf.png)

From the Figure, we conclude that Zipcodes 10011,10023,1004 have the
most revenue by quarter 1 against the total cost

We will now use Percent Units as a key metric.This will tell us that
which zipcode has maximum properties.

    dff <- plot_my_graph(Percent_units) # call the ordering and filtering function using Percent units as key metric

    dff$zipcode <- factor(dff$zipcode, levels = unique(dff$zipcode)[order(dff$Percent_units, decreasing = TRUE)])
    dff %>%  plot_ly(x = ~zipcode, y = ~Percent_units, type = 'scatter',mode = 'markers', size = ~Percent_units, marker = list(color = c('green','blue','red','grey','grey','grey','grey','grey','grey','grey'))) %>%
      layout(title = "Top zipcodes with maximum percent of properties",
             xaxis = list(title = "Zipcodes"),
             yaxis = list(title = "Percent"))

    ## Warning: `line.width` does not currently support multiple values.

![unnamed-chunk-21-1](https://user-images.githubusercontent.com/39188654/51153317-aa417900-183d-11e9-8e8e-74a81716086d.png)

From the above two plots, we can say that the zipcodes with maximum
percent of properties are not the ones which generate the maximum
revenue by quarter1 against total cost. Meaning, revenue vs cost for a
zipcode and total number of properties in a zipcode are not closely
correlated . To take this further, we use the below logic.

Revenue by Q1 vs Zipcodes
=========================

    ##### same function with different value of n=2000. This will ask the ordering function(plot_my_graph2) to check and order the top 2000 records , which will cover information of almost every zipcode given information of revenue in q1.

    plot_my_graph2 <- function(col_name){
      # sort the dataframe for by col_name in descending order and subset for top N zipcodes
      
      v <- enquo(col_name)
      
      n = 2000  
      df_sorted_unique_id <- arrange(summary_df[summary_df$id>10,],desc(!!v)) [1:n,] # order data
      
      # reassign factor levels
      
      df_sorted_unique_id$zipcode <- factor(df_sorted_unique_id$zipcode)
      
      
      # Return zipcodes
      
      return(df_sorted_unique_id)
      
    }


    plot_my_graph2(Revenue_by_q1) %>%  plot_ly( x = ~zipcode, y = ~Revenue_by_q1, type = 'bar')%>%
      layout(title = "Top zipcodes by Revenue in Quarter 1",
             xaxis = list(title = "Zipcode"),
             yaxis = list(title = "Revenue"))

    ## Warning: Ignoring 762 observations

![unnamed-chunk-22-1](https://user-images.githubusercontent.com/39188654/51153323-b299b400-183d-11e9-8dff-ef096dead2b3.png)
This message shows that even if n = 1238, then too all the zipcodes
would have been queried against revenue.

From the plot above, we can see that the top 4 profitable zipcodes as
per revenue by q1 are 10011, 10023,10014 and 10013 . An important stat
to notice is that 10011,10023 and 10014 are the most profitable zipcodes
as per the first plot (revenue vs cost) . So, if the investors are
interested in buying properties which cost less but have higher returns
then 10011,10014 and 10023 are the best choice.Even 10013 ranks 5th in
terms of revenue in q1 but it comes with a little higher cost.Revenue by
Year vs Zipcode as a metric also leads to similar finding.

To be very clear on our findings, lets take another metric , Revenue by
Cost Ratio.

    plot_my_graph2(Cost_by_Revenue) %>%  plot_ly( x = ~zipcode, y = ~Cost_by_Revenue, type = 'bar', marker = list(color = c('green','grey','grey','grey','grey','grey','grey','grey','grey','grey'))) %>%
      layout(title = "Number of Years to break even",
             xaxis = list(title = "Zipcodes"),
             yaxis = list(title = "Cost by Revenue Ratio"))

    ## Warning: Ignoring 762 observations

![unnamed-chunk-23-1](https://user-images.githubusercontent.com/39188654/51153329-ba595880-183d-11e9-962c-4468fb779e66.png)
Conclusion
==========

Zipcodes 10011,10023,10014,10013,10003 and 10036 have substantial number
of properties with high roi. These zipcodes also have substantial number
of properties listed.

Executive Summary
=================

The main agenda of the analysis was to find out top zipcodes for a real
estate company that wants to buy properties in New York city and put
them on rent to have high short term gains. AirBnb and ZillowData were
the datasets available for analysis. In simple words, we forecasted the
property cost data for particular zipcodes in Zillow Data and compared
them with the revenue data (which we generated using R-modeling). This
comparison gives us insights on potential profitability , given that the
forecasted values are close to the actual price.

We finally concluded that investment worthy zipcodes should not only
have more number of properties but also a good return on
investment.\*\*From graphs, it was inferred that zipcodes
10011,10023,10014,1003 and 10036 is agood mix of zipcodes which includes
high number of properties and high return on investment too.
