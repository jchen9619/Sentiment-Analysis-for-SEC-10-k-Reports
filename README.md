<img src="https://www.dnv.nl/Images/image%20leaflet%201000x500_tcm10-185922.jpg">

# Sentiment Analysis for SEC 10-k Reports
View Code

## Table of Contents
- [Project Objective](#project-objective)
- [Data Source](#data-source) 
- [Text Extraction and Cleaning](#text-extraction-and-cleaning) 
- [Comparison of Sentiment Word Frequency](#comparison-of-sentiment-word-frequency)  
- [Identification of Top Sentiment Words](#identification-of-top-sentiment-words)  
- [Methods Used](#methods-used)
- [Technologies](#technologies)

## Project Objective
[(Back to top)](#table-of-contents)
<br>
Sentiment analysis has been increasingly popular in finance as text processing allows us to generate valuable insight without reading the entirety of lengthy documents. Leveraging multiple natural language processing techniques, this project compares the overall sentiment of fiscal year 2021 10-k financial reports for five pharmaceutical companies: Pfizer Inc., Moderna, Inc., Johnson & Johnson, Eli Lilly and Company and AbbVie Inc, based on text in items 1A. Risk Factors and 7. Management's Discussion and Analysis of Financial Condition and Results of Operations. The project contains three sections.

1. Text Extraction and Cleaning: Reports were converted from HTML to PDFs with PDFkit. The entire document was then parsed to extract items 1A and 7 with PyPDF and RegEx key word search function. After turning all words into lower case, stop words (e.g. “a”, “the”, “is”, “are”, etc. ) and punctuations were removed to lighten the content that needed to be processed, as they do not add meaningful information to a sentence.
The following initial steps were performed to prepare the reports for further analysis: <br>
- Convert HTML to PDFs with PDFKit
- Extract items 1A and 7 PyPDF and RegEx key word search
- Transform all words to lower case
- Remove punctuations and stop words (non-meaningful words that do not add much information to a sentence, e.g. “a”, “the”, “is”, “are”, etc.) 
2. Comparison of Sentiment Word Frequency: This section culminates in a table that shows total positive and negative sentiment words as absolute numbers and as a percentage of total non-stop-words. This is done in the scope of Item 1A, Item 7 and these two sections combined. 
3. Identification of Top Sentiment Words: For each company, the top 10 most frequent sentiment words in items 1A and 7 are displayed. Searching these words in their respective sections produce meaningful insight on a companies' financial and operational developments in the fiscal year. 

## Data Source
[(Back to top)](#table-of-contents)
<br>
**10k Reports for FY2021 from Securities and Exchange Commission (SEC):** <br>
- [Pfizer Inc.](http://www.sec.gov/Archives/edgar/data/0000078003/000007800322000027/pfe-20211231.htm) <br>
- [Moderna, Inc.](http://www.sec.gov/Archives/edgar/data/1682852/000168285222000012/mrna-20211231.htm) <br>
- [Johnson and Johnson](http://www.sec.gov/Archives/edgar/data/200406/000020040622000022/jnj-20220102.htm) <br>
- [Eli Lilly and Company](http://www.sec.gov/Archives/edgar/data/59478/000005947822000068/lly-20211231.htm) <br>
- [AbbVie Inc.](http://www.sec.gov/Archives/edgar/data/0001551152/000155115222000007/abbv-20211231.htm) <br>

**Dictionary of Sentiment Words:** <br>
- [Loughran-McDonald Master Dictionary w/ Sentiment Word Lists](https://sraf.nd.edu/loughranmcdonald-master-dictionary/)

## Text Extraction and Cleaning
[(Back to top)](#table-of-contents)
<br>
The following table displays the amount of stop words and punctuations removed from the original text:





## Comparison of Sentiment Word Frequency
[(Back to top)](#table-of-contents)
<br>
tokenized

## Identification of Top Sentiment Words
[(Back to top)](#table-of-contents)
<br>

## Methods Used
[(Back to top)](#table-of-contents)
<br>
- HTML to PDF Conversion
- Text Extraction by Key Word Search
- Lower case transformation
- Stop word removal
- Punctuation removal
- Tokenization
- Word count

## Technologies
[(Back to top)](#table-of-contents)
<br>
- Python
- PDFKit
- PyPDF
- Pandas
- RegEx
- NLTK
- Spacy
- IPython

