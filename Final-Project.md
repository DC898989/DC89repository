<center> 
     <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="300"> 
</center>

# Assignment: Notebook for Peer Assignment
Estimated time needed: 45 minutes


# Assignment Scenario

Congratulations! You have just been hired by a US Venture Capital firm as a data analyst.

The company is considering foreign grain markets to help meet its supply chain requirements for its recent investments in the microbrewery and microdistillery industry, which is involved with the production and distribution of craft beers and spirits.

Your first task is to provide a high level analysis of crop production in Canada. Your stakeholders want to understand the current and historical performance of certain crop types in terms of supply and price volatility. For now they are mainly interested in a macro-view of Canada's crop farming industry, and how it relates to the relative value of the Canadian and US dollars.


# Introduction

Using this R notebook you will:

1.  Understand four datasets 
2.  Load the datasets into four separate tables in a Db2 database
3.  Execute SQL queries unsing the RODBC R package to answer assignment questions 

You have already encountered two of these datasets in the previous practice lab. You will be able to reuse much of the work you did there to prepare your database tables for executing SQL queries.


# Understand the datasets

To complete the assignment problems in this notebook you will be using subsetted snapshots of two datasets from Statistics Canada, and one from the Bank of Canada. The links to the prepared datasets are provided in the next section; the interested student can explore the landing pages for the source datasets as follows:

1.  <a href="https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMRP0203ENSkillsNetwork890-2022-01-01&pid=3210035901">Canadian Principal Crops (Data & Metadata)</a>
2.  <a href="https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMRP0203ENSkillsNetwork890-2022-01-01&pid=3210007701">Farm product prices (Data & Metadata)</a>
3.  <a href="https://www.bankofcanada.ca/rates/exchange/daily-exchange-rates?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMRP0203ENSkillsNetwork890-2022-01-01">Bank of Canada daily average exchange rates</a>


### 1. Canadian Principal Crops Data *

This dataset contains agricultural production measures for the principle crops grown in Canada, including a breakdown by province and teritory, for each year from 1908 to 2020.

For this assignment you will use a preprocessed snapshot of this dataset (see below).

A detailed description of this dataset can be obtained from the StatsCan Data Portal at:
https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3210035901  
Detailed information is included in the metadata file and as header text in the data file, which can be downloaded - look for the 'download options' link.  

### 2. Farm product prices

This dataset contains monthly average farm product prices for Canadian crops and livestock by province and teritory, from 1980 to 2020 (or 'last year', whichever is greatest).

For this assignment you will use a preprocessed snapshot of this dataset (see below).

A description of this dataset can be obtained from the StatsCan Data Portal at:
https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3210007701 
The information is included in the metadata file, which can be downloaded - look for the 'download options' link.  

### 3. Bank of Canada daily average exchange rates *

This dataset contains the daily average exchange rates for multiple foreign currencies. Exchange rates are expressed as 1 unit of the foreign currency converted into Canadian dollars. It includes only the latest four years of data, and the rates are published once each business day by 16:30 ET.

For this assignment you will use a snapshot of this dataset with only the USD-CAD exchange rates included (see next section). We have also prepared a monthly averaged version which you will be using below.

A brief description of this dataset and the original dataset can be obtained from the Bank of Canada Data Portal at:
https://www.bankofcanada.ca/rates/exchange/daily-exchange-rates/

( * these datasets are the same as the ones you used in the practice lab)


### Dataset URLs

  1.  Annual Crop Data: https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Annual_Crop_Data.csv 

  2.  Farm product prices: https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Monthly_Farm_Prices.csv
  
  3.  Daily FX Data: https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Daily_FX.csv
  
  4.  Monthly FX Data: https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Monthly_FX.csv
  

<span style="color:red">**IMPORTANT:**</span> You will be loading these datasets directly into R data frames from these URLs instead of from the StatsCan and Bank of Canada portals. The versions provided at these URLs are simplified and subsetted versions of the original datasets.


#### Now let's load these datasets into four separate Db2 tables.
Let's first load the RODBC package:



```R
library(RODBC)
```

## Problem 1
#### Create tables
Establish a connection to the Db2 database, and create the following four tables using the RODBC package in R. 
Use the separate cells provided below to create each of your tables.

1.  **CROP_DATA**
2.  **FARM_PRICES**
3.  **DAILY_FX**
4.  **MONTHLY_FX**  

The previous practice lab will help you accomplish this.


### Solution 1



```R
# Establish database connection
dsn_driver <- "{IBM DB2 ODBC Driver}"
dsn_database <- "..."
dsn_hostname <- "..."
dsn_port <- "..."
dsn_protocol <- "..."
dsn_uid <- "..."
dsn_pwd <- "..." 
dsn_security <- "ssl"

conn_path <- paste("DRIVER=",dsn_driver,
              	";DATABASE=",dsn_database,
              	";HOSTNAME=",dsn_hostname,
              	";PORT=",dsn_port,
              	";PROTOCOL=",dsn_protocol,
              	";UID=",dsn_uid,
              	";PWD=",dsn_pwd,
              	";SECURITY=",dsn_security,   	 
                	sep="")
conn <- odbcDriverConnect(conn_path)
conn
```


    RODBC Connection 1
    Details:
      case=nochange
      DRIVER={IBM DB2 ODBC DRIVER}
      UID=...
      PWD...
      DATABASE=bludb
      HOSTNAME=...
      PORT=...
      PROTOCOL=TCPIP
      SECURITY=SSL



```R
# CROP_DATA:
df1 <- sqlQuery(conn, 
                    "CREATE TABLE CROP_DATA (
                                      CD_ID INTEGER NOT NULL,
                                      YEAR DATE NOT NULL,
                                      CROP_TYPE VARCHAR(20) NOT NULL,
                                      GEO VARCHAR(20) NOT NULL, 
                                      SEEDED_AREA INTEGER NOT NULL,
                                      HARVESTED_AREA INTEGER NOT NULL,
                                      PRODUCTION INTEGER NOT NULL,
                                      AVG_YIELD INTEGER NOT NULL,
                                      PRIMARY KEY (CD_ID)
                                      )", 
                    errors=FALSE
                    )

    if (df1 == -1){
        cat ("An error has occurred.\n")
        msg <- odbcGetErrMsg(conn)
        print (msg)
    } else {
        cat ("Table was created successfully.\n")
    }
```

    Table was created successfully.



```R
# FARM_PRICES:
df2 <- sqlQuery(conn,
                	"CREATE TABLE FARM_PRICES (
                                  	CD_ID INTEGER NOT NULL,
                                  	DATE DATE NOT NULL,
                                  	CROP_TYPE VARCHAR(20) NOT NULL,
                                  	GEO VARCHAR(20) NOT NULL,
                                  	PRICE_PRERMT DECIMAL NOT NULL,
                                  	PRIMARY KEY (CD_ID)
                                  	)"
                    errors=FALSE
                    )

    if (df2 == -1){
        cat ("An error has occurred.\n")
        msg <- odbcGetErrMsg(conn)
        print (msg)
    } else {
        cat ("Table was created successfully.\n")
    }
```

    Table was created successfully.



```R
# DAILY_FX:
df3 <- sqlQuery(conn,
                	"CREATE TABLE DAILY_FX (
                                  	DFX_ID INTEGER NOT NULL,
                                  	DATE DATE NOT NULL,
                                  	FXUSDCAD FLOAT(6) NOT NULL,
                                  	PRIMARY KEY (DFX_ID)
                                  	)"
```

    Table was created successfully.



```R
# MONTHLY_FX:
df4 <- sqlQuery(conn,
                	"CREATE TABLE MONTHLY_FX (
                                  	DFX_ID INTEGER NOT NULL,
                                  	DATE DATE NOT NULL,
                                  	FXUSDCAD FLOAT(6) NOT NULL,
                                  	PRIMARY KEY (DFX_ID)
                                  	)",
                	errors=FALSE
                	)

	if (df4 == -1){
    	shcat ("An error has occurred.\n")
    	msg <- odbcGetErrMsg(conn)
    	print (msg)
	} else {
    	cat ("Table was created successfully.\n")
	}
```

    Table was created successfully.



```R
sql.info <- sqlTypeInfo(conn)
conn.info <- odbcGetInfo(conn)
conn.info["DBMS_Name"]
conn.info["DBMS_Ver"]
conn.info["Driver_ODBC_Ver"]
```


<strong>DBMS_Name:</strong> 'DB2/LINUXX8664'



<strong>DBMS_Ver:</strong> '11.05.0800'



<strong>Driver_ODBC_Ver:</strong> '03.51'


## Problem 2
#### Read Datasets and Load Tables
Read the datasets into R dataframes using the urls provided above. Then load your tables.


###  Solution 2



```R
crop_df <- read.csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Annual_Crop_Data.csv', colClasses=c(YEAR="character"))
farm_df <- read.csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Monthly_Farm_Prices.csv', colClasses=c(DATE="character"))
monthly_df <- read.csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Monthly_FX.csv', colClasses=c(DATE="character"))
daily_df <- read.csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-RP0203EN-SkillsNetwork/labs/Final%20Project/Daily_FX.csv', colClasses=c(DATE="character"))

head(crop_df)
head(farm_df)
head(monthly_df)
head(daily_df)
```


<table>
<caption>A data.frame: 6 × 8</caption>
<thead>
	<tr><th></th><th scope=col>CD_ID</th><th scope=col>YEAR</th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>SEEDED_AREA</th><th scope=col>HARVESTED_AREA</th><th scope=col>PRODUCTION</th><th scope=col>AVG_YIELD</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>0</td><td>1965-12-31</td><td>Barley</td><td>Alberta     </td><td>1372000</td><td>1372000</td><td>2504000</td><td>1825</td></tr>
	<tr><th scope=row>2</th><td>1</td><td>1965-12-31</td><td>Barley</td><td>Canada      </td><td>2476800</td><td>2476800</td><td>4752900</td><td>1920</td></tr>
	<tr><th scope=row>3</th><td>2</td><td>1965-12-31</td><td>Barley</td><td>Saskatchewan</td><td> 708000</td><td> 708000</td><td>1415000</td><td>2000</td></tr>
	<tr><th scope=row>4</th><td>3</td><td>1965-12-31</td><td>Canola</td><td>Alberta     </td><td> 297400</td><td> 297400</td><td> 215500</td><td> 725</td></tr>
	<tr><th scope=row>5</th><td>4</td><td>1965-12-31</td><td>Canola</td><td>Canada      </td><td> 580700</td><td> 580700</td><td> 512600</td><td> 885</td></tr>
	<tr><th scope=row>6</th><td>5</td><td>1965-12-31</td><td>Canola</td><td>Saskatchewan</td><td> 224600</td><td> 224600</td><td> 242700</td><td>1080</td></tr>
</tbody>
</table>




<table>
<caption>A data.frame: 6 × 5</caption>
<thead>
	<tr><th></th><th scope=col>CD_ID</th><th scope=col>DATE</th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>PRICE_PRERMT</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>0</td><td>1985-01-01</td><td>Barley</td><td>Alberta     </td><td>127.39</td></tr>
	<tr><th scope=row>2</th><td>1</td><td>1985-01-01</td><td>Barley</td><td>Saskatchewan</td><td>121.38</td></tr>
	<tr><th scope=row>3</th><td>2</td><td>1985-01-01</td><td>Canola</td><td>Alberta     </td><td>342.00</td></tr>
	<tr><th scope=row>4</th><td>3</td><td>1985-01-01</td><td>Canola</td><td>Saskatchewan</td><td>339.82</td></tr>
	<tr><th scope=row>5</th><td>4</td><td>1985-01-01</td><td>Rye   </td><td>Alberta     </td><td>100.77</td></tr>
	<tr><th scope=row>6</th><td>5</td><td>1985-01-01</td><td>Rye   </td><td>Saskatchewan</td><td>109.75</td></tr>
</tbody>
</table>




<table>
<caption>A data.frame: 6 × 3</caption>
<thead>
	<tr><th></th><th scope=col>DFX_ID</th><th scope=col>DATE</th><th scope=col>FXUSDCAD</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>0</td><td>2017-01-01</td><td>1.319276</td></tr>
	<tr><th scope=row>2</th><td>1</td><td>2017-02-01</td><td>1.310726</td></tr>
	<tr><th scope=row>3</th><td>2</td><td>2017-03-01</td><td>1.338643</td></tr>
	<tr><th scope=row>4</th><td>3</td><td>2017-04-01</td><td>1.344021</td></tr>
	<tr><th scope=row>5</th><td>4</td><td>2017-05-01</td><td>1.360705</td></tr>
	<tr><th scope=row>6</th><td>5</td><td>2017-06-01</td><td>1.329805</td></tr>
</tbody>
</table>




<table>
<caption>A data.frame: 6 × 3</caption>
<thead>
	<tr><th></th><th scope=col>DFX_ID</th><th scope=col>DATE</th><th scope=col>FXUSDCAD</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>0</td><td>2017-01-03</td><td>1.3435</td></tr>
	<tr><th scope=row>2</th><td>1</td><td>2017-01-04</td><td>1.3315</td></tr>
	<tr><th scope=row>3</th><td>2</td><td>2017-01-05</td><td>1.3244</td></tr>
	<tr><th scope=row>4</th><td>3</td><td>2017-01-06</td><td>1.3214</td></tr>
	<tr><th scope=row>5</th><td>4</td><td>2017-01-09</td><td>1.3240</td></tr>
	<tr><th scope=row>6</th><td>5</td><td>2017-01-10</td><td>1.3213</td></tr>
</tbody>
</table>




```R
sqlSave(conn, crop_df, "CROP_DATA", append=TRUE, fast=FALSE, rownames=FALSE, colnames=FALSE, verbose=FALSE)
sqlSave(conn, farm_df, "FARM_PRICES", append=TRUE, fast=FALSE, rownames=FALSE, colnames=FALSE, verbose=FALSE)
sqlSave(conn, monthly_df, "MONTHLY_FX", append=TRUE, fast=FALSE, rownames=FALSE, colnames=FALSE, verbose=FALSE)
sqlSave(conn, daily_df, "DAILY_FX", append=TRUE, fast=FALSE, rownames=FALSE, colnames=FALSE, verbose=FALSE)
```


```R
sqlSave(conn, crop_df, "CROP_DATA", append=TRUE, fast=FALSE, rownames=FALSE, colnames=FALSE, verbose=FALSE)
```

## Now execute SQL queries using the RODBC R package to solve the assignment problems.

## Problem 3
#### How many records are in the farm prices dataset?


### Solution 3



```R
query = "SELECT COUNT(CD_ID)FROM FARM_PRICES"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 1</caption>
<thead>
	<tr><th></th><th scope=col>1</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>2678</td></tr>
</tbody>
</table>



## Problem 4
#### Which geographies are included in the farm prices dataset?


### Solution 4



```R
query = "SELECT GEO FROM FARM_PRICES GROUP BY GEO"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 2 × 1</caption>
<thead>
	<tr><th></th><th scope=col>GEO</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Alberta     </td></tr>
	<tr><th scope=row>2</th><td>Saskatchewan</td></tr>
</tbody>
</table>



## Problem 5
#### How many hectares of Rye were harvested in Canada in 1968?


### Solution 5



```R
query = "SELECT SUM(HARVESTED_AREA) AS CANADA_HARVESTED_RYE_1968 FROM CROP_DATA
WHERE GEO = 'Canada' AND CROP_TYPE = 'Rye' AND YEAR(YEAR) = 1968"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 1</caption>
<thead>
	<tr><th></th><th scope=col>CANADA_HARVESTED_RYE_1968</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>274100</td></tr>
</tbody>
</table>



## Problem 6
#### Query and display the first 6 rows of the farm prices table for Rye.


### Solution 6



```R
query = "SELECT * FROM FARM_PRICES 
WHERE CROP_TYPE ='Rye' LIMIT 6"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 6 × 5</caption>
<thead>
	<tr><th></th><th scope=col>CD_ID</th><th scope=col>DATE</th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>PRICE_PRERMT</th></tr>
	<tr><th></th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td> 4</td><td>1985-01-01</td><td>Rye</td><td>Alberta     </td><td>100</td></tr>
	<tr><th scope=row>2</th><td> 5</td><td>1985-01-01</td><td>Rye</td><td>Saskatchewan</td><td>109</td></tr>
	<tr><th scope=row>3</th><td>10</td><td>1985-02-01</td><td>Rye</td><td>Alberta     </td><td> 95</td></tr>
	<tr><th scope=row>4</th><td>11</td><td>1985-02-01</td><td>Rye</td><td>Saskatchewan</td><td>103</td></tr>
	<tr><th scope=row>5</th><td>16</td><td>1985-03-01</td><td>Rye</td><td>Alberta     </td><td> 96</td></tr>
	<tr><th scope=row>6</th><td>17</td><td>1985-03-01</td><td>Rye</td><td>Saskatchewan</td><td>106</td></tr>
</tbody>
</table>



## Problem 7
#### Which provinces grew Barley? 


### Solution 7



```R
query = "SELECT GEO FROM CROP_DATA 
WHERE CROP_TYPE='Barley'GROUP BY GEO"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 3 × 1</caption>
<thead>
	<tr><th></th><th scope=col>GEO</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Alberta     </td></tr>
	<tr><th scope=row>2</th><td>Canada      </td></tr>
	<tr><th scope=row>3</th><td>Saskatchewan</td></tr>
</tbody>
</table>



## Problem 8
#### Find the first and last dates for the farm prices data.


### Solution 8


```R
query = "SELECT min(DATE), max(DATE)
	FROM FARM_PRICES"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 2</caption>
<thead>
	<tr><th></th><th scope=col>1</th><th scope=col>2</th></tr>
	<tr><th></th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;date&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>1985-01-01</td><td>2020-12-01</td></tr>
</tbody>
</table>



## Problem 9
#### Which crops have ever reached a farm price greater than or equal to &#0036;350 per metric tonne?


### Solution 9



```R
query = "SELECT DISTINCT (CROP_TYPE), PRICE_PRERMT, YEAR(DATE) 
AS YEAR FROM FARM_PRICES 
WHERE PRICE_PRERMT >= 350 ORDER BY PRICE_PRERMT"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 397 × 3</caption>
<thead>
	<tr><th></th><th scope=col>CROP_TYPE</th><th scope=col>PRICE_PRERMT</th><th scope=col>YEAR</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Canola</td><td>350</td><td>1985</td></tr>
	<tr><th scope=row>2</th><td>Canola</td><td>350</td><td>1995</td></tr>
	<tr><th scope=row>3</th><td>Canola</td><td>350</td><td>2003</td></tr>
	<tr><th scope=row>4</th><td>Canola</td><td>350</td><td>2007</td></tr>
	<tr><th scope=row>5</th><td>Canola</td><td>351</td><td>1999</td></tr>
	<tr><th scope=row>6</th><td>Canola</td><td>351</td><td>2007</td></tr>
	<tr><th scope=row>7</th><td>Canola</td><td>352</td><td>1997</td></tr>
	<tr><th scope=row>8</th><td>Canola</td><td>352</td><td>1998</td></tr>
	<tr><th scope=row>9</th><td>Canola</td><td>352</td><td>2003</td></tr>
	<tr><th scope=row>10</th><td>Canola</td><td>352</td><td>2007</td></tr>
	<tr><th scope=row>11</th><td>Canola</td><td>353</td><td>2002</td></tr>
	<tr><th scope=row>12</th><td>Canola</td><td>353</td><td>2003</td></tr>
	<tr><th scope=row>13</th><td>Canola</td><td>353</td><td>2007</td></tr>
	<tr><th scope=row>14</th><td>Canola</td><td>354</td><td>1985</td></tr>
	<tr><th scope=row>15</th><td>Canola</td><td>354</td><td>1988</td></tr>
	<tr><th scope=row>16</th><td>Canola</td><td>354</td><td>2004</td></tr>
	<tr><th scope=row>17</th><td>Canola</td><td>355</td><td>1998</td></tr>
	<tr><th scope=row>18</th><td>Canola</td><td>356</td><td>2003</td></tr>
	<tr><th scope=row>19</th><td>Canola</td><td>357</td><td>1994</td></tr>
	<tr><th scope=row>20</th><td>Canola</td><td>357</td><td>2004</td></tr>
	<tr><th scope=row>21</th><td>Canola</td><td>358</td><td>1994</td></tr>
	<tr><th scope=row>22</th><td>Canola</td><td>358</td><td>1995</td></tr>
	<tr><th scope=row>23</th><td>Canola</td><td>358</td><td>1997</td></tr>
	<tr><th scope=row>24</th><td>Canola</td><td>358</td><td>2007</td></tr>
	<tr><th scope=row>25</th><td>Canola</td><td>359</td><td>1985</td></tr>
	<tr><th scope=row>26</th><td>Canola</td><td>359</td><td>1995</td></tr>
	<tr><th scope=row>27</th><td>Canola</td><td>359</td><td>1996</td></tr>
	<tr><th scope=row>28</th><td>Canola</td><td>360</td><td>1997</td></tr>
	<tr><th scope=row>29</th><td>Canola</td><td>361</td><td>1996</td></tr>
	<tr><th scope=row>30</th><td>Canola</td><td>362</td><td>1995</td></tr>
	<tr><th scope=row>⋮</th><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><th scope=row>368</th><td>Canola</td><td>569</td><td>2012</td></tr>
	<tr><th scope=row>369</th><td>Canola</td><td>570</td><td>2012</td></tr>
	<tr><th scope=row>370</th><td>Canola</td><td>571</td><td>2011</td></tr>
	<tr><th scope=row>371</th><td>Canola</td><td>574</td><td>2008</td></tr>
	<tr><th scope=row>372</th><td>Canola</td><td>576</td><td>2012</td></tr>
	<tr><th scope=row>373</th><td>Canola</td><td>577</td><td>2011</td></tr>
	<tr><th scope=row>374</th><td>Canola</td><td>578</td><td>2012</td></tr>
	<tr><th scope=row>375</th><td>Canola</td><td>585</td><td>2012</td></tr>
	<tr><th scope=row>376</th><td>Canola</td><td>586</td><td>2012</td></tr>
	<tr><th scope=row>377</th><td>Canola</td><td>588</td><td>2012</td></tr>
	<tr><th scope=row>378</th><td>Canola</td><td>589</td><td>2012</td></tr>
	<tr><th scope=row>379</th><td>Canola</td><td>592</td><td>2008</td></tr>
	<tr><th scope=row>380</th><td>Canola</td><td>593</td><td>2012</td></tr>
	<tr><th scope=row>381</th><td>Canola</td><td>594</td><td>2008</td></tr>
	<tr><th scope=row>382</th><td>Canola</td><td>594</td><td>2012</td></tr>
	<tr><th scope=row>383</th><td>Canola</td><td>595</td><td>2012</td></tr>
	<tr><th scope=row>384</th><td>Canola</td><td>597</td><td>2013</td></tr>
	<tr><th scope=row>385</th><td>Canola</td><td>600</td><td>2013</td></tr>
	<tr><th scope=row>386</th><td>Canola</td><td>602</td><td>2012</td></tr>
	<tr><th scope=row>387</th><td>Canola</td><td>611</td><td>2013</td></tr>
	<tr><th scope=row>388</th><td>Canola</td><td>614</td><td>2013</td></tr>
	<tr><th scope=row>389</th><td>Canola</td><td>616</td><td>2013</td></tr>
	<tr><th scope=row>390</th><td>Canola</td><td>618</td><td>2013</td></tr>
	<tr><th scope=row>391</th><td>Canola</td><td>630</td><td>2013</td></tr>
	<tr><th scope=row>392</th><td>Canola</td><td>631</td><td>2013</td></tr>
	<tr><th scope=row>393</th><td>Canola</td><td>638</td><td>2013</td></tr>
	<tr><th scope=row>394</th><td>Canola</td><td>639</td><td>2013</td></tr>
	<tr><th scope=row>395</th><td>Canola</td><td>640</td><td>2013</td></tr>
	<tr><th scope=row>396</th><td>Canola</td><td>646</td><td>2013</td></tr>
	<tr><th scope=row>397</th><td>Canola</td><td>651</td><td>2013</td></tr>
</tbody>
</table>



## Problem 10
#### Rank the crop types harvested in Saskatchewan in the year 2000 by their average yield. Which crop performed best? 


### Solution 10



```R
query = "SELECT CROP_TYPE AS CROP_RANK, MAX(AVG_YIELD)AS AVG_YIELD_YEAR_2000 FROM CROP_DATA 
WHERE YEAR(YEAR)=2000 AND GEO='Saskatchewan' 
GROUP BY CROP_TYPE ORDER BY AVG_YIELD_YEAR_2000 desc"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 4 × 2</caption>
<thead>
	<tr><th></th><th scope=col>CROP_RANK</th><th scope=col>AVG_YIELD_YEAR_2000</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Barley</td><td>2800</td></tr>
	<tr><th scope=row>2</th><td>Wheat </td><td>2200</td></tr>
	<tr><th scope=row>3</th><td>Rye   </td><td>2100</td></tr>
	<tr><th scope=row>4</th><td>Canola</td><td>1400</td></tr>
</tbody>
</table>




```R
# Which crop performed best?
query = "SELECT CROP_TYPE AS BEST_PERFORMING_CROP, AVG_YIELD AS TOP_YIELD FROM CROP_DATA 
WHERE YEAR(YEAR)=2000 AND GEO='Saskatchewan' LIMIT 1"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 2</caption>
<thead>
	<tr><th></th><th scope=col>BEST_PERFORMING_CROP</th><th scope=col>TOP_YIELD</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Barley</td><td>2800</td></tr>
</tbody>
</table>



## Problem 11
#### Rank the crops and geographies by their average yield (KG per hectare) since the year 2000. Which crop and province had the highest average yield since the year 2000? 


### Solution 11



```R
query = "SELECT CROP_TYPE, GEO, AVG_YIELD FROM CROP_DATA
WHERE YEAR(YEAR) >=2000
ORDER BY AVG_YIELD DESC LIMIT 10"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 10 × 3</caption>
<thead>
	<tr><th></th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>AVG_YIELD</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Barley</td><td>Alberta</td><td>4100</td></tr>
	<tr><th scope=row>2</th><td>Barley</td><td>Alberta</td><td>4100</td></tr>
	<tr><th scope=row>3</th><td>Barley</td><td>Alberta</td><td>3980</td></tr>
	<tr><th scope=row>4</th><td>Wheat </td><td>Alberta</td><td>3900</td></tr>
	<tr><th scope=row>5</th><td>Barley</td><td>Alberta</td><td>3900</td></tr>
	<tr><th scope=row>6</th><td>Wheat </td><td>Alberta</td><td>3900</td></tr>
	<tr><th scope=row>7</th><td>Barley</td><td>Canada </td><td>3900</td></tr>
	<tr><th scope=row>8</th><td>Barley</td><td>Alberta</td><td>3890</td></tr>
	<tr><th scope=row>9</th><td>Barley</td><td>Canada </td><td>3820</td></tr>
	<tr><th scope=row>10</th><td>Barley</td><td>Canada </td><td>3810</td></tr>
</tbody>
</table>




```R
# Which crop and province had the highest average yield since the year 2000?

query = "SELECT CROP_TYPE, GEO, AVG_YIELD FROM CROP_DATA
WHERE YEAR(YEAR) >=2000
ORDER BY AVG_YIELD DESC LIMIT 1"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 3</caption>
<thead>
	<tr><th></th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>AVG_YIELD</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Barley</td><td>Alberta</td><td>4100</td></tr>
</tbody>
</table>



## Problem 12
#### Use a subquery to determine how much wheat was harvested in Canada in the most recent year of the data.


### Solution 12



```R
query = "SELECT MAX(YEAR) AS MOST_RECENT_YEAR, SUM(HARVESTED_AREA) AS TOT_HARVESTED_WHEAT_CANADA FROM CROP_DATA
WHERE YEAR(YEAR)=(SELECT MAX(YEAR(YEAR))FROM CROP_DATA 
WHERE GEO = 'Canada' AND CROP_TYPE  = 'Wheat')"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 1 × 2</caption>
<thead>
	<tr><th></th><th scope=col>MOST_RECENT_YEAR</th><th scope=col>TOT_HARVESTED_WHEAT_CANADA</th></tr>
	<tr><th></th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>2020-12-31</td><td>38897100</td></tr>
</tbody>
</table>



## Problem 13
#### Use an implicit inner join to calculate the monthly price per metric tonne of Canola grown in Saskatchewan in both Canadian and US dollars. Display the most recent 6 months of the data.


### Solution 13



```R
query = "SELECT A.DATE,A.CROP_TYPE,A.GEO,A.PRICE_PRERMT AS CANADIAN_DOLLARS, 
(A.PRICE_PRERMT / B.FXUSDCAD) AS US_DOLLARS, B.FXUSDCAD 
FROM FARM_PRICES A , MONTHLY_FX B 
WHERE YEAR(A.DATE) = YEAR(B.DATE) 
AND MONTH(A.DATE) = MONTH(B.DATE) 
AND CROP_TYPE ='Canola' 
AND GEO = 'Saskatchewan' 
ORDER BY DATE DESC LIMIT 6"
sqlQuery(conn,query)
```


<table>
<caption>A data.frame: 6 × 6</caption>
<thead>
	<tr><th></th><th scope=col>DATE</th><th scope=col>CROP_TYPE</th><th scope=col>GEO</th><th scope=col>CANADIAN_DOLLARS</th><th scope=col>US_DOLLARS</th><th scope=col>FXUSDCAD</th></tr>
	<tr><th></th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>2020-12-01</td><td>Canola</td><td>Saskatchewan</td><td>507</td><td>395.8553</td><td>1.280771</td></tr>
	<tr><th scope=row>2</th><td>2020-11-01</td><td>Canola</td><td>Saskatchewan</td><td>495</td><td>378.7821</td><td>1.306820</td></tr>
	<tr><th scope=row>3</th><td>2020-10-01</td><td>Canola</td><td>Saskatchewan</td><td>474</td><td>358.6912</td><td>1.321471</td></tr>
	<tr><th scope=row>4</th><td>2020-09-01</td><td>Canola</td><td>Saskatchewan</td><td>463</td><td>350.0125</td><td>1.322810</td></tr>
	<tr><th scope=row>5</th><td>2020-08-01</td><td>Canola</td><td>Saskatchewan</td><td>464</td><td>350.9290</td><td>1.322205</td></tr>
	<tr><th scope=row>6</th><td>2020-07-01</td><td>Canola</td><td>Saskatchewan</td><td>462</td><td>342.2602</td><td>1.349850</td></tr>
</tbody>
</table>




```R

```

## Author(s)

<h4> Jeff Grossman </h4>

## Contributor(s)

<h4> Rav Ahuja </h4>

## Change log

| Date       | Version | Changed by    | Change Description                                                                                          |
| ---------- | ------- | ------------- | ----------------------------------------------------------------------------------------------------------- |
| 2021-04-01 | 0.7     | Jeff Grossman | Split Problem 1 solution cell into multiple cells, fixed minor bugs |
| 2021-03-12 | 0.6     | Jeff Grossman | Cleaned up content for production |
| 2021-03-11 | 0.5     | Jeff Grossman | Moved more advanced problems to optional honours module |
| 2021-03-10 | 0.4     | Jeff Grossman | Added introductory and intermediate level problems and removed some advanced problems |
| 2021-03-04 | 0.3     | Jeff Grossman | Moved some problems to a new practice lab as prep for this assignment
| 2021-03-04 | 0.2     | Jeff Grossman | Sorted problems roughly by level of difficulty and relegated more advanced ones to ungraded bonus problems  |
| 2021-02-20 | 0.1     | Jeff Grossman | Started content creation                                                                                    |


## <h3 align="center"> © IBM Corporation 2021. All rights reserved. <h3/>



```R
####Disconnect From DataBase
```


```R
close(conn)
```
