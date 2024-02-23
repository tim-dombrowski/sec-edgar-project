# SEC EDGAR Project

### Overview

This project will aim to explore the [edgar R package](https://cran.r-project.org/package=edgar), which is a wrapper for the [SEC's EDGAR API](https://www.sec.gov/edgar/sec-api-documentation). Using this package, we can programmatically download filing data from the SEC's EDGAR database. Some additional tools within the package allow for computing sentiment measures following the popular Loughran and McDonald Sentiment Master Dictionary. This project will demonstrate how to use the edgar package to download and parse 10-K filings, and then compute sentiment measures for the filings.

### Additional Context Around the Package

Below are some references for academic papers that relate to this edgar R package and the textual analysis tools used to compute sentiment measures. These papers provide some additional context for the package and the tools used in this project.

* [Lonare, Patil, and Raut (2021), "edgar: An R package for the U.S. SEC EDGAR retrieval and parsing of corporate filings," *SoftwareX*, Volume 16.](https://doi.org/10.1016/j.softx.2021.100865)
* [Loughran and McDonald (2011), "When is a Liability not a Liability? Textual Analysis, Dictionaries, and 10-Ks," *Journal of Finance*, 66:1, 35-65.](https://ssrn.com/abstract=1331573)
* [Bodnaruk, Loughran, and McDonald (2015), "Using 10-K Text to Gauge Financial Constraints, *Journal of Financial and Quantitative Analysis*," 50:4, 1-24.](https://ssrn.com/abstract=2331544)
* [Loughran and McDonald (2016), "Textual Analysis in Accounting and Finance: A Survey, *Journal of Accounting Research*, 54:4,1187-1230.](https://ssrn.com/abstract=2504147)

### Repository Structure

The data work for this project demo is contained in the R Notebook directory of this repository. On GitHub, the webpage within that folder should display the README.md file, which contains the compiled output of the R Notebook. If you wish to explore the source code locally, then you can open the secedgar.Rmd file in RStudio and execute the code chunks to replicate the data work. Note the `output: html_notebook` line in the header of that file, which indicates that the R Markdown document is an R Notebook. 

After running the code chunks in RStudio and making any desired changes, you can then create a copy that will generate a copy that will appear on GitHub. To do this, save a copy of the R Notebook and name it README.Rmd. Then, change the header line for the output type to `output: github_document`. This will switch the file from being an R Notebook to an R Markdown file that will compile into a generic [Markdown](https://www.markdownguide.org/) file (.md). This format (along with the README name) will automatically be recognized by GitHub and displayed in-browser. This will also replace the Preview button with an option to Knit the Markdown file. This knitting process will re-run all the code chunks and generate a new README.md file inside of the R Notebook folder, which will display on GitHub.


