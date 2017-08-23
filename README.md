# usaspend
## This is a Zeppelin notebook used for analysis on correlation between reported Annual Revenue, Obligated Funds, and reported Executive Pay for recipients of U.S. Federal funding (tax payer money). Uses Scala Spark.
** Data Source: ** USASpending.gov

USAspending.gov is the publicly accessible, searchable website mandated by the Federal Funding Accountability and Transparency Act of 2006 to give the American public access to information on how their tax dollars are spent.

In California, total funds awarded:
* FY 2017:     $239,293,120,061
* FY 2016:     $289,921,076,736
* FY 2015:     $261,466,060,785

For the purpose of this study, we look at FY 2015 data only.

### Motivations

* Currently no comprehensive study on executive compensation as it relates to profits on government contracts.
  * Data set is large enough that it requires distributed system to perform analysis. 
  * Tried running on a 4 Core MacBook Pro with 16 GB of memory, with jobs split among 4 cores
  * Each job run took 2-3 minutes to complete
  * Ran into memory leaks on complex calculations (getting % of compensation/annualRevenue)
* Spark in MapReduce: Cluster EMR setup with Spark
  * Each job run took anywhere between 10-58 seconds to complete
  * Loading Spark DataFrames and Parquet files allow more flexible/computationally efficient querying


