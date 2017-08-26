## Annual Revenue Vs. Executive Pay for Recipients of U.S. Federal Funds

The JSON file is a Zeppelin notebook used for analysis on correlation between reported Annual Revenue, Obligated Funds, and reported Executive Pay for recipients of U.S. Federal funding (tax payer money). The project uses Scala Spark to query data. The data source used is USASpending.gov.

USAspending.gov is the publicly accessible, searchable website mandated by the Federal Funding Accountability and Transparency Act of 2006 to give the American public access to information on how their tax dollars are spent.

California is the greatest recipient of such federal fund. In California, total funds awarded:
* FY 2017:     $239,293,120,061
* FY 2016:     $289,921,076,736
* FY 2015:     $261,466,060,785

For the purpose of this study, we look at FY 2015 data only.

### Motivation

* Currently no comprehensive study on executive compensation as it relates to profits on government contracts.
  * Data set is large enough that it requires distributed system to perform analysis. 
  * Tried running on a 4 Core MacBook Pro with 16 GB of memory, with jobs split among 4 cores
  * Each job run took 2-3 minutes to complete
  * Ran out of memory in doing complex calculations (getting % of compensation/annualRevenue)
* Spark in MapReduce: Cluster EMR setup with Spark
  * Each job run took anywhere between 10-58 seconds to complete
  * Loading Spark DataFrames and Parquet files allow more flexible/computationally efficient querying. (Parquet is Spark’s preferred serialization format. Has efficient built-in compression and encoding scheme, on a per-column level.)

ETL steps – 90% of work in data analysis.


### Questions

* What companies have the most federal dollars obligated?
  * Lockheed Martin Coporation
  * Raytheon Company
  * The Boeing Company
  * McKesson Corporation
  * Northrop Grumman
* What companies have the highest self-reported annual revenue?
* What companies have the highest executive compensation?
* What companies have the highest executive payout as a percentage of total revenue?
* What companies have the highest executive payout as a percentage of sum of dollars obligated?

For the purposes of this project, this analysis is limited to FY 2015 data only,  8.71 GB, which I partition into Parquet format.


### Cluster Setup

* EMR-5.8.0 with Spark: Spark 2.2.0 on Hadoop 2.7.3 YARN with Ganglia 3.7.2 and Zeppelin 0.7.2
* m4.4xlarge
* 3 Nodes: 1 Master, 2 core nodes


### ETL Steps

The data was downloaded as a batched CSV file from USASpending.gov. The CSV file contained a record of all federal funding allocation to companies and non-profits in 2015.

Of the 225 fields in the original data, we are most interested in dollarsobligated, vendorname, annualrevenue, prime_awardee_executive1_compensation, prime_awardee_executive2_compensation, prime_awardee_executive3_compensation, prime_awardee_executive4_compensation, and prime_awardee_executive5_compensation.

The original CSV file had 261 line breaks within individual data fields out of over 20M rows of data. The CSV could not be loaded into a Spark Dataframe with this inconsistent formatting. The problematic rows were removed from the data set for this exercise. Here are the steps I took:

1. Extract CSV data from source. 

2. Apply transformations for data cleansing– data was originally formatted in CSV, but line breaks within fields prevented the CSV from being converted into Parquet.

   a. Select for lines that begin and end with a quote.

   ``` pv Data_Feed.csv | grep "^\".*\"\r" > data_feed_good.csv  ```

   b. Filter out lines that begin with an end-quote followed by a comma.
   
   ``` pv data_feed_good.csv | grep -v "^\",.*\r" > data_feed_good_2.csv ```

Note:  These data cleansing steps remove 0.001305% of data.

3. Convert data to Parquet.
4. Upload data to Amazon S3 bucket.

   ``` aws s3 cp --rec clean_data.parquet/ s3://sarah-usaspendingfy2015/clean_data.parquet ```

5.  Load the data into Spark Data Frame

    ``` val df = sqlContext.read.parquet("s3://sarah-usaspendingfy2015/clean_data.parquet") ```




### Some General Analysis

* Out of the 196630 reported companies, only 1458 have reported over $1M for executive compensation and annual revenue.
* Only 4536 of 154596 companies have executive compensation above $0. 
* 799 companies report executive compensation over $1M.
* Annual Revenue field is a required field to for publicly traded companies, but not all companies report or report accurately. 
* Prime Awardee ExecutiveX Compensation is a denormalized view of the top 5 executive’s pay for each company. Not all companies report.
* Need to deal with duplicate and differing vendor-reported fields.
* Need to deal with mismatched vendor names like “BOEING COMPANY, THE” and “THE BOEING COMPANY”.
  * May use Machine Learning approach

