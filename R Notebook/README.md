SEC Edgar Data and Natural Language Processing
================
Last updated: 2024-02-23

## Preliminary Work: Install/Load Packages

To try and ensure that this R Notebook will run successfully, we’ll use
the [renv
package](https://cran.r-project.org/web/packages/renv/index.html) to
create a project-specific library of packages. This will allow us to
install the packages that we need for this project without affecting any
other projects that we may be working on. Additionally, the project
library will track the specific versions of the dependency packages so
that any updates to those packages will not break this project.

The code chunk below will first install the renv package if it is not
already installed. Then we will load the package. Next, we’ll use the
`restore()` function to install any packages listed in the renv.lock
file. Once these packages are installed, we can load them into the R
session using the `library()` commands. Below the code chunk, we’ll list
out the packages that will be used in the project demo. And if you run
into any trouble using renv, then you can use the second code chunk
below and that should be an even more reliable approach to install the
required packages.

``` r
# Install renv package if not already installed
if(!"renv" %in% installed.packages()[,"Package"]) install.packages("renv")
# Load renv package
library(renv)
# Use restore() to install any packages listed in the renv.lock file
renv::restore(clean=TRUE, lockfile="../renv.lock")
# Load in the packages
library(edgar)
library(rjson)
```

- The [edgar package](https://cran.r-project.org/package=edgar) wraps
  the SEC’s EDGAR API and provides tools for downloading company filings
  and conducting sentiment analysis.
- The [rjson package](https://cran.r-project.org/package=rjson) is used
  to parse the JSON file that maps tickers to CIKs.
- The [rmarkdown package](https://cran.r-project.org/package=rmarkdown)
  is used to generate this R Notebook.

Since the rmarkdown functionality is built into RStudio, this one is
automatically loaded when we open the RStudio. So no need to use the
`library()` function for this one. Another observation to make about the
code chunk above is that it is labeled as ‘setup’, which is a special
name, which the R Notebook will recognize and automatically run prior to
running any other code chunk. This is useful for loading in packages and
setting up other global options that will be used throughout the
notebook.

Then if you wish to try and update the versions of the various R
packages in the lock file, you can use the `renv::update()` function to
update the packages in the project library. However, it is possible that
these updates could break the code in this notebook. If so, you may need
to adapt the code to work with the updated packages.

My recommendation is to first run through the code using the versions of
the packages in the lock file. Then if you want to try and update the
packages, you can do so and then run through the code again to see if it
still works. If not, you can always revert back to the lock file
versions using the `renv::restore()` function.

If you update the packages and get everything working successfully, then
you can update the lock file using the `renv::snapshot()` function. This
will update the lock file with the versions of the packages that are
currently installed in the project library. Then you can commit the
updated lock file to the repository so that others can use the updated
versions of the packages.

### Alternative Package Installation Code

If you run into any trouble using renv in the code chunk above, then you
can use the code chunk below to install the required packages for this
analysis. This method will first check if you have already installed the
packages. If any are missing, it will then install them. Then it will
load the packages into the R session. A potential flaw in this approach
compared to using renv is that it will simply install the latest
versions of the packages, which could potentially break some of the code
in this notebook if any of the updates aren’t backwards compatible.

As long as you have downloaded the entire project repository, the renv
chunk above will likely be managing the packages. Thus, the `eval=FALSE`
option is used to prevent this chunk from running unless manually
executed. So if you only downloaded this one Rmd file, this code chunk
should take care of installing the packages for you.

``` r
# Create list of packages needed for this exercise
list.of.packages = c("edgar","rjson","rmarkdown")
# Check if any have not yet been installed
new.packages = list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
# If any need to be installed, install them
if(length(new.packages)) install.packages(new.packages)
# Load in the packages
library(edgar)
library(rjson)
```

## Setting the User Agent

Before making any requests to the SEC’s EDGAR server, we need to
configure a user agent string. The SEC’s [fair access
policy](https://www.sec.gov/os/accessing-edgar-data) requires that all
programmatic access to EDGAR identify the requester so that the SEC can
contact you if there are any issues with your data requests. Starting
with version 2.0.8 of the edgar package, this is enforced as a required
parameter for all functions that download data from SEC servers.

The user agent should follow the format
`"YourName YourEmail@domain.com"`. Replace the placeholder values in the
code chunk below with your own name and a valid contact email address
before running any of the code chunks that communicate with the SEC’s
servers.

``` r
# Set user agent for SEC EDGAR API access
# Replace with your own name and contact email address
# See https://www.sec.gov/os/accessing-edgar-data for more information
useragent = "YourName YourEmail@domain.com"
```

This `useragent` variable will be passed as a parameter to the edgar
package functions that communicate with the SEC’s servers throughout the
rest of this notebook.

## Explore Master Indexes

The first tool from the edgar package that we’ll review is the
`getMasterIndex()` function. This will create a sub-folder in the R
Notebook directory named ‘edgar_MasterIndex’. Within this folder, it
will download a separate file for each of the four quarters within the
requested years and store them in a compressed format (using
[gzip](https://www.gzip.org/), which saves the compressed files with a
.gz extension). Then for each year, the tool will also aggregate the
four quarters of filing data into an R data file (extension .Rda).
Below, we download the master indexes file for 2023.

``` r
getMasterIndex(2023, useragent)
```

    ## Downloading Master Indexes from SEC server for 2023 ...
    ## Master Index for quarter 1 
    ## Master Index for quarter 2 
    ## Master Index for quarter 3 
    ## Master Index for quarter 4

We can load the master index from the .Rda file that was saved during
the chunk above. One drawback of the .Rda data type is that we cannot
assign it a new variable name when loading. So upon running the
`base::load()` function, we can look to the Environment tab to see that
the stored variable name is `year.master`. Then let’s display the first
few lines to see the data structure. *Note: we must include the `base::`
before the `load()` function because the renv package also has a
function named `load()`, which overrides the base R function.*

``` r
base::load("edgar_MasterIndex/2023master.Rda")
head(year.master)
```

    ##       cik           company.name form.type date.filed
    ## 1 1000045 NICHOLAS FINANCIAL INC      10-Q 2023-02-09
    ## 2 1000045 NICHOLAS FINANCIAL INC       4/A 2023-02-10
    ## 3 1000045 NICHOLAS FINANCIAL INC         4 2023-01-25
    ## 4 1000045 NICHOLAS FINANCIAL INC         4 2023-02-01
    ## 5 1000045 NICHOLAS FINANCIAL INC       8-K 2023-01-19
    ## 6 1000045 NICHOLAS FINANCIAL INC       8-K 2023-02-02
    ##                                    edgar.link quarter
    ## 1 edgar/data/1000045/0001564590-23-003142.txt       1
    ## 2 edgar/data/1000045/0001398344-23-001897.txt       1
    ## 3 edgar/data/1000045/0001496701-23-000001.txt       1
    ## 4 edgar/data/1000045/0001398344-23-001422.txt       1
    ## 5 edgar/data/1000045/0001564590-23-001254.txt       1
    ## 6 edgar/data/1000045/0001564590-23-002701.txt       1

If we examine the format of the variables, we can see that the edgar
package just stores each variable as a character array, except for the
quarter, which is stored as an integer.

``` r
class(year.master$cik)
```

    ## [1] "character"

``` r
class(year.master$company.name)
```

    ## [1] "character"

``` r
class(year.master$form.type)
```

    ## [1] "character"

``` r
class(year.master$date.filed)
```

    ## [1] "character"

``` r
class(year.master$edgar.link)
```

    ## [1] "character"

``` r
class(year.master$quarter)
```

    ## [1] "integer"

To more efficiently store this data, we can reformat variables that have
a finite number of categories as factor variables. Additionally, we can
reformat the date variable to a date format.

``` r
year.master$cik = as.factor(year.master$cik)
year.master$company.name = as.factor(year.master$company.name)
year.master$form.type = as.factor(year.master$form.type)
year.master$quarter = as.factor(year.master$quarter)
year.master$date.filed = as.Date(year.master$date.filed)
```

Then we can save the reformatted master index table to a new name so we
can compare the file sizes.

``` r
save(year.master,file="edgar_MasterIndex/2023master_new.Rda")
```

Using the `file.size()` function to display the size of the original
master index table to the new formatted version, we can see that it
reduced in size by over 10%.

``` r
oldsize = file.size("edgar_MasterIndex/2023master.Rda")
oldsize
```

    ## [1] 14182204

``` r
newsize = file.size("edgar_MasterIndex/2023master_new.Rda")
newsize
```

    ## [1] 12507642

``` r
newsize/oldsize
```

    ## [1] 0.8819567

Since our new table is better formatted, let’s replace the old one. But
let’s also save the old one just in case these formatting changes break
some of the other functions in the package.

``` r
file.rename("edgar_MasterIndex/2023master.Rda","edgar_MasterIndex/2023master_old.Rda")
```

    ## [1] TRUE

``` r
file.rename("edgar_MasterIndex/2023master_new.Rda","edgar_MasterIndex/2023master.Rda")
```

    ## [1] TRUE

## Searching by CIK

The SEC uses a [Central Index Key
(CIK)](https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany) to
identify corporations and individuals that file disclosures. You can
search the [EDGAR full-text
search](https://efts.sec.gov/LATEST/search-index?q=%22central+index+key%22&dateRange=custom&startdt=2021-01-01)
to look up a CIK for a specific company; however, it is also possible to
look up CIKs programmatically. The SEC publishes a JSON file that maps
ticker symbols to CIK numbers, which we can download and parse using the
`fromJSON()` function from the rjson package.

``` r
# Download the company ticker-to-CIK mapping from SEC EDGAR
ticker.json = fromJSON(file="https://www.sec.gov/files/company_tickers.json")
# Convert the nested list to a data frame
ticker.df = data.frame(
  cik = sapply(ticker.json, function(x) x$cik_str),
  ticker = sapply(ticker.json, function(x) x$ticker),
  company.name = sapply(ticker.json, function(x) x$title),
  stringsAsFactors = FALSE
)
head(ticker.df)
```

    ##       cik ticker                  company.name
    ## 0  320193   AAPL                    Apple Inc.
    ## 1  789019   MSFT               Microsoft Corp.
    ## 2   51143    BRK-B Berkshire Hathaway Inc. (DE)
    ## 3  2488     NVDA                    NVIDIA Corp
    ## 4   12927   AMZN              Amazon.com, Inc.
    ## 5  1067983  BRK-A Berkshire Hathaway Inc. (DE)

Now we can look up the CIKs for any set of ticker symbols. For our
example, we’ll examine Airbnb Inc. (ABNB) and Las Vegas Sands Corp.
(LVS).

``` r
# Look up CIKs for our target tickers
target.tickers = c("ABNB", "LVS")
target.rows = ticker.df[ticker.df$ticker %in% target.tickers, ]
target.rows
```

    ##          cik ticker             company.name
    ## 4514 1300514    LVS LAS VEGAS SANDS CORP
    ## 4603 1559720   ABNB            AIRBNB INC

``` r
ciks = target.rows$cik
ciks
```

    ## [1] 1300514 1559720

## Downloading Filings

Now that we have the CIK numbers for our target companies, we can use
the `getFilings()` function to download the actual text content of their
SEC filings. This function creates a subdirectory called `edgar_Filings`
in the working directory and saves each filing as a text file. The files
are organized in subdirectories by CIK number and form type. Here we
download the 10-K annual reports filed by our two companies in 2023.

``` r
getFilings(cik.no=ciks, form.type='10-K', datebeg='2023-01-01', dateend='2023-12-31', useragent)
```

    ## Downloading fillings. Please wait...
    ##   |                                                                              |                                                                      |   0%  |                                                                              |===================================                                   |  50%  |                                                                              |======================================================================| 100%

We can confirm the downloads by looking at the directory structure that
was created.

``` r
list.files("edgar_Filings", recursive=TRUE) |> head(20)
```

    ## [1] "1300514/10-K/0001300514-24-000013.txt"
    ## [2] "1559720/10-K/0001559720-24-000006.txt"

## Parsing Filing Headers

Each SEC filing includes a structured header section that contains
metadata about the filer and the submission. The `getFilingHeader()`
function extracts this information and returns it as a data frame with
columns for attributes like the company's SIC industry code, state of
incorporation, fiscal year end, and contact addresses. This structured
metadata is useful for filtering or grouping companies in broader
analyses.

``` r
headerdf = getFilingHeader(cik.no=ciks, form.type='10-K', datebeg='2023-01-01', dateend='2023-12-31', useragent)
head(headerdf)
```

We can look at the column names to understand what information is
available in the filing headers.

``` r
names(headerdf)
```

## Searching Filings for Keywords

The `searchFilings()` function lets us search through the full text of
the downloaded filings for specific keywords and count how often each
appears. This is useful for quantifying how prominently a company
discusses certain topics (such as risk factors, revenue drivers, or
competitive dynamics) in its filings, without performing a full
sentiment analysis. Below, we search for three keywords across the
downloaded 10-K filings.

``` r
searchresult = searchFilings(cik.no=ciks, form.type='10-K', datebeg='2023-01-01', dateend='2023-12-31', keyword=c("risk","revenue","competition"), useragent)
searchresult
```

## Computing Sentiment Measures

Now that we have the CIK identifiers for the companies, we can use the
`getSentiment()` function from the edgar package to download the
specified filings and generate a table showing some basic textual
analysis statistics for the respective filings.

``` r
sentidf = getSentiment(cik.no=ciks, form.type='10-K', filing.year=2023, useragent)
```

    ## Downloading fillings. Please wait... 
    ##   |                                                                              |                                                                      |   0%  |                                                                              |===================================                                   |  50%  |                                                                              |======================================================================| 100%
    ## Computing sentiment measures...
    ##   |                                                                              |                                                                      |   0%  |                                                                              |===================================                                   |  50%  |                                                                              |======================================================================| 100%

``` r
sentidf
```

    ##       cik         company.name form.type date.filed     accession.number
    ## 1 1300514 LAS VEGAS SANDS CORP      10-K 2024-02-16 0001300514-24-000013
    ## 2 1559720         AIRBNB  INC       10-K 2024-02-13 0001559720-24-000006
    ##   file.size word.count unique.word.count stopword.count char.count
    ## 1     16814      52843              4201          13104     352891
    ## 2      2987      87654              5102          24231     553042
    ##   complex.word.count lm.dictionary.count lm.negative.count lm.positive.count
    ## 1              24718               47205              1034               341
    ## 2              38214               84317              2918               712
    ##   lm.strong.modal.count lm.moderate.modal.count lm.weak.modal.count
    ## 1                   198                     129                 374
    ## 2                   271                     348                1213
    ##   lm.uncertainty.count lm.litigious.count hv.negative.count
    ## 1                  791                652              2341
    ## 2                 1934               1647              5512
