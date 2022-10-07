# A Link between Amazon Reviews and Sales Rank of Products

<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Amazon Review Analysis Report" data-toc-modified-id="Amazon Review Analysis Report"><span class="toc-item-num">1&nbsp;&nbsp;</span>Amazon Review Analysis Report</a></span><ul class="toc-item"><li><span><a href="#Author(s)" data-toc-modified-id="Author(s)-1.1"><span class="toc-item-num">1.1&nbsp;&nbsp;</span>Author(s)</a></span></li><li><span><a href="#Purpose" data-toc-modified-id="Purpose-1.2"><span class="toc-item-num">1.2&nbsp;&nbsp;</span>Purpose</a></span></li><li><span><a href="#Methodology" data-toc-modified-id="Methodology-1.3"><span class="toc-item-num">1.3&nbsp;&nbsp;</span>Methodology</a></span></li><li><span><a href="#Results" data-toc-modified-id="Results-1.4"><span class="toc-item-num">1.4&nbsp;&nbsp;</span>Results</a></span></li><li><span><a href="#Suggested-next-steps" data-toc-modified-id="Suggested-next-steps-1.5"><span class="toc-item-num">1.5&nbsp;&nbsp;</span>Suggested next steps</a></span></li></ul></li><li><span><a href="#Data Dictionary" data-toc-modified-id="Data Dictionary-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Data Dictionary</a></span></li><li><span><a href="#Dataset Identification" data-toc-modified-id="Dataset Identification-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Dataset Identification</a></span></li><li><span><a href="#References" data-toc-modified-id="References-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>References</a></span></li></ul></div>

## Purpose
 
This notebook lists the steps and results of the exploration, analysis and discussion of the dataset on Amazon reviews and their effects on sales ranks. Our team seeks to understand the relationship between a product's reviews and its effect on the product's sales on Amazon.
 
## Methodology
 
#### Our team is making the following assumptions:
- A product's sales rank is an efficient indicator of the sales quantity for the product.
 
#### Cleaning and Processing Metadata dataset
The first step we took was to clean and process our Metadata dataset. The following steps were taken to clean the metadata dataset in Excel:
 
- There was a large number of empty rows caused by HTML tables affecting the csv file. This caused many rows to be generated for just one row of actual data. Such rows were removed, and the relevant information was placed in its correct cell.
- Problem entries were removed or changed. Some empty cells had empty square brackets instead of being empty. All of these cells were converted to empty cells. Product descriptions were modified also to be easily processed in SQL & python. Starting characters such as special characters were removed.
- Ranks were amended to not include commas or any other special characters and be purely numeral based. Miscategorized products/rank rows were removed (73 out of 32981 entries).
- Prices were cleaned up. The dollar amounts were converted to floats for easier handling. Any problem entries such as irrelevant text strings were removed.
- The following columns were removed as they were either not relevant to our question, or were too sparse to use:
rownum;
category;
tech1;
fit;
tech2;
feature;
details
main_cat;
similar_item;
date;
imageURL;
imageURLHighRes.
 
 
The following steps were taken to prepare the metadata dataset:
 
- ASINs & Products Table Creation: After loading MetaData in Python, we ran .info() and .nunique on a dataframe and discovered that 352 ASINs in metadata were not unique. Such rows with non-unique ASINs were removed, as ASIN is the primary key for this dataset. The first row with the unique asin was retained and any subsequent rows with such an asin was removed. (.drop_duplicates(subset ="asin", keep = False, inplace = True) was used to do this). 704 non-unique rows were dropped.
- Of the remaining rows, all had non-empty ranks, and almost all had unique titles. A total of 31512 unique rows/products were retained in the Metadata dataset.
- Click on the following link to download the zipped folder containing the python & csv files where the data was cleaned & processed:
- https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41578
- The python script for the review and summary sentiment analysis is also included in this folder.
 
 
#### Cleaning and Processing Reviews dataset
The following steps were taken to clean the Reviews dataset, listed by columns that needed amending:
- ASIN: ‘asin’ column values were all converted to text, as some were coded as integers.
- overall: ‘overall’ values were all converted to integer
- vote: blank votes were coded as text instead of integer- there were changed to integer type
 
The next step was to pre-process the Reviews data set and run a sentiment analysis on the reviews in python. The steps to do so were as follows:
 
- The reviews were tokenized into a list of words using word_tokenize from nltk package.
- Stop words were removed from the tokenized reviews.
- A sentiment score was calculated for each review.
- The sentiment scores were added to the dataframe as a new column, named review_polarity.
- The summary column elements were tokenized into a list of words using word_tokenize from nltk package.
- Stop words were removed from the tokenized summaries.
- A sentiment score was calculated for each summary.
- The sentiment scores for summaries were added to the dataframe as a new column, named summary_polarity.
- Click on the following link to download the zipped folder containing the python & csv files where the data was cleaned & processed (same link as given in previous section):
- https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41578
- The python script in the above link also contains the python script for conducting the sentiment analysis on the review and summary texts.
 
#### Uploading Data to AWS RDS and Accessing the Database
The cleaned, processed and updated dataset (which included the sentiment scores for the reviews and summaries) was uploaded as a table into AWS RDS instance, as was the metadata dataset.
Our next step was to combine the two tables. We achieved this by using inner join to combine the two tables on the ASIN (product ID), which was the primary key for metadata and a foreign key for the reviews dataset.In order to analyze the data, we used psycopg2 library to connect our notebook to the database and run queries to pull data. The sales rank, review polarity, summary polarity and average overall rating of the product were pulled from the database and explored in depth. To download the python scipt (with PSQL commands) to load the data to AWS - click here:
- HTML file: https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41579
- Jupyter Notebook file: https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41582
 
#### Steps to Analyze Data
 
We employed statistical tools such as correlations to get an idea of the relationships between the x variables. A question we wanted to explore was if there was a difference in the effect verified reviews had on the sales rank, as opposed to all reviews (verified+unverified). We found that there was no significant difference after filtering out non-verified reviews (a subquery+where clauses were used to filter the data before joining the tables). Due to this result, we continued the rest of our exploration on the full dataset which included both verified and unverified reviews.
It is worth noting that in our analysis, a lower magnitude for sales rank is a positive thing. That is to say, if a variable has a negative correlation with sales rank, it is effectively having a positive effect on said rank, as a rank of 1 is better than a rank of 2. We found that one such variable was the number of reviews- it had a high negative correlation with rank (meaning it had a high effect on improving the product's rank standing). Two other such variables were average time since reviews had started to be posted on the product, and the average number of votes a product's reviews got. Some other correlations we found in our data existed between # reviews & helpful votes as well as summary polarity & overall review rating.
In order to further understand the relationship of these variables with the sales rank, our team employed a regression model. The x variables included in the model are average unix time, overall review score, number of helpful votes, average review polarity, average summary polarity and the number of reviews received.
We found that:
- Each of the mentioned metrics (average unix time, overall review score, number of helpful votes, average review polarity, average summary polarity and the number of reviews received) has a significant effect on product sales rank
- However, other metrics could improve rank predictions (Our model has a low R-squared)

## Full Analysis Jupyter Notebook
- To download the Jupyter notebook with the full analysis of all of the data - go to: 
- HTML file: https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41580
- Jupyter Notebook file: https://learn.bcit.ca/d2l/lor/viewer/view.d2l?ou=6605&loIdentId=41581
- This notebook includes all scatterplots, regression analysis, database queries, plots and analysis at each step.
 
 
## Results
 
In order to increase sales rank for an Amazon seller we recommend the following actions:
- Market share typically increases over time - start online presence (Amazon or other) as soon as possible
- Encourage customers to vote on whether reviews were helpful
- Encourage customers to give reviews on products
- Follow up on reviews with negative summary sentiment or low overall ratings
 
 
## Suggested next steps
It would be interesting to next examine the same features/metrics but divide the reviews into positive and negative reviews when doing the analysis. The questions to answer: what are the key features that affect the product sales rank for negative reviews? What are the key features that affect sales rank for positive reviews? Does verified have an effect on positive or negative reviews also?

# Data Identification
Our team is using two datasets. The first lists the reviews for products, reviewer's ID, product ID number, name of reviewer, overall rating etc. In this dataset, we are concerened with the review, rating and the product ID the said reviews are associated with. 
The second dataset contains metadata regarding products. It includes product ID (which links it to our previous dataset), name of the product, it's brand and most importantly, the sales rank information for the product. 
By combining information from both these datasets, using the product ID number as our reference to connect the two datasets, we seek to explore the effects of reviews left by customers on a product to it's sales ranking.


  
### Amazon Review Data- All Beauty
 
 * Title: not available
 * URI: https://nijianmo.github.io/amazon/index.html
 * Keywords: Reviews, Vote, Overall
 * Publication Date: 2018
 * Publisher: UCSD
 * Creator: Julian McAuley, Jianmo Ni
 * Contact Point: N/A
 * Spatial Coverage: not available
 * Temporal Coverage: May 1996- October 2018
 * Language: not available
 * Date & Time Formats: unixTime
 * Data Version: not available
 * Access Date: Jan 21, 2022
 
### Amazon Meta Data- All Beauty
* Title: not available
 * URI: https://nijianmo.github.io/amazon/index.html
 * Keywords: Product, Sales Rank
 * Publication Date: 2018
 * Publisher: UCSD
 * Creator: Julian McAuley, Jianmo Ni
 * Contact Point: N/A
 * Spatial Coverage: not available
 * Temporal Coverage: October 2018
 * Language: not available
 * Date & Time Formats: N/A
 * Data Version: not available
 * Access Date: Jan 21, 2022

# References
List relevant references.

1. Notebook sharing guidelines from reproducible-science-curriculum: https://reproducible-science-curriculum.github.io/publication-RR-Jupyter/
2. Guide for developing shareable notebooks by Kevin Coakley, SDSC: https://github.com/kevincoakley/sharing-jupyter-notebooks/raw/master/Jupyter-Notebooks-Sharing-Recommendations.pdf
3. Guide for sharing notebooks by Andrea Zonca, SDSC: https://zonca.dev/2020/09/how-to-share-jupyter-notebooks.html
4. Jupyter Notebook Best Practices: https://towardsdatascience.com/jupyter-notebook-best-practices-f430a6ba8c69
5. Introduction to Jupyter templates nbextension: https://towardsdatascience.com/stop-copy-pasting-notebooks-embrace-jupyter-templates-6bd7b6c00b94  
    5.1. Table of Contents (Toc2) readthedocs: https://jupyter-contrib-nbextensions.readthedocs.io/en/latest/nbextensions/toc2/README.html  
    5.2. Steps to install toc2: https://stackoverflow.com/questions/23435723/installing-ipython-notebook-table-of-contents
6. Rule A, Birmingham A, Zuniga C, Altintas I, Huang SC, et al. (2019) Ten simple rules for writing and sharing computational analyses in Jupyter Notebooks. PLOS Computational Biology 15(7): e1007007. https://doi.org/10.1371/journal.pcbi.1007007. Supplementary materials: example notebooks (https://github.com/jupyter-guide/ten-rules-jupyter) and tutorial (https://github.com/ISMB-ECCB-2019-Tutorial-AM4/reproducible-computational-workflows)
7. Languages supported by Jupyter kernels: https://github.com/jupyter/jupyter/wiki/Jupyter-kernels
8. EarthCube notebooks presented at EC Annual Meeting 2020: https://www.earthcube.org/notebooks
