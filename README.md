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

1. Text Extraction and Cleaning: <br>
The following initial steps were performed to prepare the reports for further analysis: <br>
- Convert HTML to PDFs with PDFKit
- Extract items 1A and 7 PyPDF and RegEx key word search
- Transform all words to lower case
- Remove punctuations and stop words (non-meaningful words that do not add much information to a sentence, e.g. “a”, “the”, “is”, “are”, etc.) 
2. Comparison of Sentiment Word Frequency: <br>
3. This section tokenizes the remaining non-stop-words, and culminates in a table that shows total positive and negative sentiment words as absolute numbers and as a percentage of total non-stop-words. This is done in the scope of Item 1A, Item 7 and these two sections combined. 
4. Identification of Top Sentiment Words: <br>
For each company, the top 10 most frequent sentiment words in items 1A and 7 are displayed. Searching these words in their respective sections produce meaningful insight on a companies' financial and operational developments in the fiscal year. 

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
The following table displays the amount of stop words and punctuations removed from the original text. 35-45% of total word count were removed for the five companies. 

  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/stopwords.png" />
</p>

## Comparison of Sentiment Word Frequency
[(Back to top)](#table-of-contents)
<br>
The following tables display the frequency of positive and negative words in item 1A, item 7 and the two sections combined. 
  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/1afreq.png" />
</p>
  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/7freq.png" />
</p>
  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/totalfreq.png" />
</p>
With the range of Positive Words Frequency being much narrower than that of Negative Words Frequency among the five companies, Negative Words Frequency serves as a differentiator in the tones of the 10-k financial reports.

Overall, Moderna, Inc. ranks highest in frequency of negative words, followed by Eli Lilly and Company, Johnson & Johnson, Pfizer Inc. and AbbVie Inc. However, for the individual items, Eli Lilly and Company ranks highest instead. It appears Moderna's high overall ranking is mostly attributable to high negative words frequency in item 1A.

Zooming in on item 7, which discusses the company's annual performance in greater detail than item 1A, Eli Lilly and Company, Pfizer Inc. and Johnson and Johnson's reports are worth a more in-depth read as their tones indicate more negative sentiments than the rest.

We will shed more light on this analysis in section II, higlighting the sentiment words that appear most frequently and the sentences they belong to.

## Identification of Top Sentiment Words
[(Back to top)](#table-of-contents)
<br>
The following tables display top 10 most frequently sentiment words for item 1A and 7 respectively. 
  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/1a%20top10.png" />
</p>
  <img src="https://github.com/jchen9619/Sentiment-Analysis-for-SEC-10-k-Reports/blob/main/img/7%20top10.png" />
</p>
You may notice one or two of the top 10 most frequent sentiment words being of the same root word (i.e. "adverse" and "adversely"). This is due to lemmatization being skipped to save significant time on running this notebook. Regardless, the output still produces top 8-9 words. <br>

Item 1A tends to include more negative words because it discusses risk factors. This is consistent with the five companies. </mark> Item 7 shows a more diverse range of sentiments, with a more even split between positive and negative words. In conjunction with the quantitative comparison of sentiment words frequency in section I, searching for the most frequent sentiment words can lead to informative insight, with the following examples: 
    
**AbbVie Inc., Item 1A** <br>
"Successful(ly)", "Adversely"<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The <mark style="background-color: #FFFF00">**successful**</mark> discovery, development, manufacturing and sale of biologics is a long, expensive and uncertain process. There are unique risks and uncertainties with biologics. For example, access to and supply of necessary biological materials, such as cell lines, may be limited and governmental regulations restrict access to and regulate the transport and use of such materials. In addition, the development, manufacturing and sale of biologics is subject to regulations that are often more complex and extensive than the regulations applicable to other pharmaceutical products...Biologics are also frequently costly to manufacture because production inputs are derived from living animal or plant material, and some biologics cannot be made synthetically. Failure to <mark style="background-color: #FFFF00">**successfully**</mark> discover, develop, manufacture and sell biologics—including Humira—could <mark style="background-color: #FFFF00">**adversely**</mark> impact AbbVie's business and results of operations.  

**Pfizer Inc., Item 1A** <br>
"Litigation", "Loss", "Adversely"<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;We recorded direct product and/or Alliance revenues of more than $1 billion for each of nine products that collectively accounted for 75% of our total revenues in 2021. In particular, Comirnaty/BNT162b2 accounted for 45% of our total revenues in 2021. For additional information, see Notes 1 and 17. If these products or any of our other major products were to experience <mark style="background-color: yellow">**loss**</mark> of patent protection (if applicable), changes in prescription or vaccination growth rates, material product liability <mark style="background-color: yellow">**litigation**</mark>, unexpected side effects or safety concerns, regulatory proceedings, negative publicity affecting doctor or patient confidence, pressure from existing competitive products, changes in labeling, pricing and access pressures or supply shortages or if a new, more effective product should be introduced, the <mark style="background-color: yellow">**adverse**</mark> impact on our revenues could be significant.
    
**Eli Lilly and Company, Item 7** <br>
"Exclusivity", "Favorable", "Loss", "Severe(ly)" <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Revenue of Alimta, a treatment for various cancers, decreased 2 percent in the U.S., driven by decreased volume, partially offset by higher realized prices. Revenue outside the U.S. decreased 22 percent, primarily driven by decreased volume due to the entry of generic competition in certain markets and, to a lesser extent, lower realized prices, partially offset by the <mark style="background-color: yellow">**favorable**</mark> impact of foreign exchange rates. Following the <mark style="background-color: yellow">**loss**</mark> of <mark style="background-color: yellow">**exclusivity**</mark> in major European countries and Japan in June 2021, we faced, and remain exposed to, generic competition which has eroded revenue and is likely to continue to rapidly and <mark style="background-color: yellow">**severely**</mark> erode revenue from current levels. In the U.S., we expect the limited entry of generic competition starting February 2022 and subsequent unlimited entry starting April 2022. We expect that the entry of generic competition following the <mark style="background-color: yellow">**loss**</mark> of <mark style="background-color: yellow">**exclusivity**</mark> in the U.S. will cause a rapid and <mark style="background-color: yellow">**severe**</mark> decline in revenue.    

**Moderna Inc., Item 7** <br>
"Advance", "Progress" <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; We expect that research and development expenses will increase in 2022 as we continue to <mark style="background-color: yellow">**progress**</mark> our indication expansion of mRNA-1273, and continue to develop our pipeline and <mark style="background-color: yellow">**advance**</mark> our product candidates into later-stage development. In addition, we also expect to incur significant costs related to the development of variantspecific COVID-19 candidates and our next-generation COVID-19 vaccine candidate (mRNA-1283).
  
**Johnson & Johnson, Item 7** <br>
"Achieved", "Positive"  <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In 2021, sales by companies in Europe <mark style="background-color: yellow">**achieved**</mark> growth of 24.3% as compared to the prior year, which included operational growth of 20.7% and a <mark style="background-color: yellow">**positive**</mark> currency impact of 3.6%. Sales by companies in the Western Hemisphere (excluding the U.S.) <mark style="background-color: yellow">**achieved**</mark> growth of 7.8% as compared to the prior year, which included operational growth of 7.3% and a <mark style="background-color: yellow">**positive**</mark> currency impact of 0.5%. Sales by companies in the Asia-Pacific, Africa region <mark style="background-color: yellow">**achieved**</mark> growth of 14.1% as compared to the prior year, including operational growth of 11.4% and a <mark style="background-color: yellow">**positive**</mark> currency impact of 2.7%. 

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

