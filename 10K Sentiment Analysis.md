## I. Text Extraction and Cleaning

####  Import libraries


```python
import pandas as pd
import pdfkit
import PyPDF2
import re
from nltk.tokenize import word_tokenize
import spacy
from string import punctuation
from IPython.display import display_html 
```

### A) Creating the Dataset

#### Compile all URLs


```python
base_url = 'http://www.sec.gov/Archives/edgar/data'
cik_ls = ['0000078003', '1682852','200406','59478','0001551152']
filings_num_ls = ['000007800322000027','000168285222000012','000020040622000022','000005947822000068',
                  '000155115222000007']
company_ls=["Pfizer Inc.", "Moderna, Inc.", "Johnson & Johnson", "Eli Lilly and Company", "AbbVie Inc."]
ticker_ls =['pfe','mrna','jnj','lly','abbv']
fye_ls = ['20211231','20211231','20220102','20211231','20211231']
```


```python
url_ls=[]
for i in range(0,5):
    url= base_url+'/'+cik_ls[i]+'/'+filings_num_ls[i]+'/'+ticker_ls[i]+'-'+fye_ls[i]+'.htm'
    url_ls.append(url)
url_ls
```




    ['http://www.sec.gov/Archives/edgar/data/0000078003/000007800322000027/pfe-20211231.htm',
     'http://www.sec.gov/Archives/edgar/data/1682852/000168285222000012/mrna-20211231.htm',
     'http://www.sec.gov/Archives/edgar/data/200406/000020040622000022/jnj-20220102.htm',
     'http://www.sec.gov/Archives/edgar/data/59478/000005947822000068/lly-20211231.htm',
     'http://www.sec.gov/Archives/edgar/data/0001551152/000155115222000007/abbv-20211231.htm']



#### Converting HTML to PDFs


```python
for i in range(0,5):
    pdfkit.from_url(url_ls[i], ticker_ls[i]+'.pdf')
```

#### Retrieve Items 1A and 7 from all PDFs

Create lookup dictionaries for function parse_section_text:


```python
toc_dict = {"pfe": 1, "mrna": 1, "jnj": 2, "lly": 1, "abbv": 2}
```


```python
toc_start_dict = {"1A": "risk factors ", 
                  "7": "management’s discussion and analysis of financial condition and results of operations ",
                 "7alt":"management’s discussion and analysis of results of operations and financial condition "}
```


```python
toc_end_dict = {"1A": "properties ",
                "7": "quantitative and qualitative disclosures about market risk ",
                "7alt": "quantitative and qualitative disclosures about market risk "}
```


```python
end_section_dict = {"1A": "item 2",
                    "7": "item 7a",
                   "7alt":"item 7a"}
```

Create function parse_section_text


```python
def parse_section_text(ticker, section):
    with open(ticker+'.pdf','rb') as pdf_file:  
    read_pdf = PyPDF2.PdfFileReader(pdf_file)        
    toc_content = read_pdf.getPage(toc_dict[ticker]).extractText().replace("\t"," ").replace("\n"," ")
                  .lower().replace("'","’")

    start_page = int(toc_content[re.search(toc_start_dict[section], toc_content).end()
                                 -1:re.search(toc_start_dict[section], toc_content).end()+3].split()[0])  
    end_page = int(toc_content[re.search(toc_end_dict[section], toc_content).end()
                                 -1:re.search(toc_end_dict[section], toc_content).end()+3].split()[0])  

    pages_to_extract=[]
    for i in range(0, read_pdf.numPages):
        try:
            last3_pg_end=read_pdf.getPage(i).extractText().replace("\t"," ").replace("\n"," ").lower().
            replace("   |    2021 form 10-k","")[-3:] 
            if int(re.sub('\D', '', last3_pg_end)) in [start_page, end_page]:
                pages_to_extract.append(i)
        except Exception:
            pass

    first_page=read_pdf.getPage(pages_to_extract[0]).extractText().replace("\t"," ").replace("\n"," ").lower()
               .replace("   |    2021 form 10-k","").replace("\"","").replace("'","’")
    first_page_keep=first_page[re.search(toc_start_dict[section], first_page).end():]  

    last_page=read_pdf.getPage(pages_to_extract[1]).extractText().replace("\t"," ").replace("\n"," ").lower()
              .replace("   |    2021 form 10-k","").replace("\"","").replace("'","’")
    last_page_keep=last_page[:re.search(end_section_dict[section], last_page).start()]

    content_str=first_page_keep
    for i in range(pages_to_extract[0]+1,pages_to_extract[1]):
        page_content=read_pdf.getPage(i).extractText().replace("\t"," ").replace("\n"," ").lower()
                     .replace("   |    2021 form 10-k","").replace("\"","").replace("'","’")
        content_str+=page_content
    content_str+=last_page_keep
    return content_str
```

Create dictionaries for item 1A and 7


```python
item1A_dict={}
for i in range(0,5):
    item1A_dict[ticker_ls[i]]= parse_section_text(ticker_ls[i],'1A')
```


```python
item7_dict={}
for i in range(0,len(ticker_ls)):
    if ticker_ls[i] in ['jnj','lly']:
        item7_dict[ticker_ls[i]] = parse_section_text(ticker_ls[i],'7alt')
    else:
        item7_dict[ticker_ls[i]] = parse_section_text(ticker_ls[i],'7')
```

Taking a look at first 100 words of items 1A and 7 for Pfizer Inc. (PFE)


```python
print("Item 1A: "+' '.join(item1A_dict['pfe'].split()[0:100]))
```

    Item 1A: this section describes the material risks to our business, which should be considered carefully in addition to the other information in this report and our other filings with the sec. investors should be aware that it is not possible to predict or identify all such factors and that the following is pfizer inc. 2021 form 10-k 13not meant to be a complete discussion of all potential risks or uncertainties. additionally, our business is subject to general risks applicable to any company, such as economic conditions, geopolitical events, extreme weather and natural disasters. if known or unknown risks or uncertainties materialize,



```python
print("Item 7: "+' '.join(item7_dict['pfe'].split()[0:100]))
```

    Item 7: overview of our performance, operating environment, strategy and outlook financial highlights the following is a summary of certain financial performance metrics (in billions, except per share data): 2021 total revenues––$81.3 billion 2021 net cash flow from operations––$32.6 billion an increase of 95% compared to 2020 an increase of 126% compared to 2020 2021 reported diluted eps––$3.85 2021 adjusted diluted eps (non-gaap)––$4.42* an increase of 137% compared to 2020 an increase of 96% compared to 2020 * for additional information regarding adjusted diluted eps (which is a non-gaap financial measure), including reconciliations of certain gaap reported to non-gaap adjusted information, see


### B) Text Cleaning

#### Stop word and punctuation removal

Create list of stop words


```python
nlp = spacy.load('en_core_web_lg')
```


```python
stop_words_ls=list(nlp.Defaults.stop_words)
```


```python
#Taking a look at first 10 elements
print(stop_words_ls[0:9])
```

    ['along', 'few', 'amongst', 'is', 'except', 'these', 'over', 'noone', 'hereupon']


Display list of punctuation


```python
punctuation
```




    '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'



Remove stop words and punctuation from dictionaries *item1A_dict* and *item7_dict*


```python
def create_nsw_dictionaries(section_dict):
    nsw_dict={}
    nsw_str_ls=[]
    for t in range(0, len(ticker_ls)):
        all_words=word_tokenize(section_dict[ticker_ls[t]])
        nsw_str=''
        for i in range(0, len(all_words)):
            if all_words[i] not in stop_words_ls and all_words[i] not in punctuation:
                nsw_str+=all_words[i]+' '
        nsw_str_ls.append(nsw_str)
        nsw_dict[ticker_ls[t]]=nsw_str_ls[t]
    return nsw_dict
```


```python
item1A_nsw_dict=create_nsw_dictionaries(item1A_dict)
item7_nsw_dict=create_nsw_dictionaries(item7_dict)
```

Table for stop words and punctuations removed:


```python
def compare_length(dict):
    len_ls=[]
    for t in range(0, len(ticker_ls)):
        len_ls.append(len(dict[ticker_ls[t]].split()))
    return len_ls
```


```python
company=pd.Series(company_ls)
item1A_orig=pd.Series(compare_length(item1A_dict))
item1A_nsw=pd.Series(compare_length(item1A_nsw_dict))
item7_orig=pd.Series(compare_length(item7_dict))
item7_nsw=pd.Series(compare_length(item7_nsw_dict))
length_df=pd.DataFrame({'Company': company, "Item 1A Original": item1A_orig, 'Item 1A Shortened': item1A_nsw,
          'Item 1A Stop Words & Punctuation': item1A_orig-item1A_nsw,
          'Item 7 Original': item7_orig, 'Item 7 Shortened': item7_nsw,
          'Item 7 Stop Words & Punctuation': item7_orig-item7_nsw, 
          'Total Stop Words & Punctuation': item1A_orig-item1A_nsw+item7_orig-item7_nsw,
           '% of Words Removed': (item1A_orig-item1A_nsw+item7_orig-item7_nsw)/(item1A_orig+item7_orig)})
length_df['% of Words Removed']=length_df['% of Words Removed'].map('{:.1%}'.format)
title_style = [dict(selector="caption",
            props=[("text-align", "center"),
                   ("font-size", "100%"),
                   ("color", 'black'),
                   ("font-weight", "bold")])]


length_df.style.hide_index().set_properties(subset=['Item 1A Stop Words & Punctuation',
                                                    'Item 7 Stop Words & Punctuation',
                                                   'Total Stop Words & Punctuation', '% of Words Removed'], 
                                                    **{'font-weight': 'bold'})
                                                    .set_caption('Text Cleaning Word Count Comparison')
                                                    .set_table_styles(title_style)
```




<style  type="text/css" >
    #T_137299cc_1209_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }#T_137299cc_1209_11ed_99db_acde48001122row0_col3,#T_137299cc_1209_11ed_99db_acde48001122row0_col6,#T_137299cc_1209_11ed_99db_acde48001122row0_col7,#T_137299cc_1209_11ed_99db_acde48001122row0_col8,#T_137299cc_1209_11ed_99db_acde48001122row1_col3,#T_137299cc_1209_11ed_99db_acde48001122row1_col6,#T_137299cc_1209_11ed_99db_acde48001122row1_col7,#T_137299cc_1209_11ed_99db_acde48001122row1_col8,#T_137299cc_1209_11ed_99db_acde48001122row2_col3,#T_137299cc_1209_11ed_99db_acde48001122row2_col6,#T_137299cc_1209_11ed_99db_acde48001122row2_col7,#T_137299cc_1209_11ed_99db_acde48001122row2_col8,#T_137299cc_1209_11ed_99db_acde48001122row3_col3,#T_137299cc_1209_11ed_99db_acde48001122row3_col6,#T_137299cc_1209_11ed_99db_acde48001122row3_col7,#T_137299cc_1209_11ed_99db_acde48001122row3_col8,#T_137299cc_1209_11ed_99db_acde48001122row4_col3,#T_137299cc_1209_11ed_99db_acde48001122row4_col6,#T_137299cc_1209_11ed_99db_acde48001122row4_col7,#T_137299cc_1209_11ed_99db_acde48001122row4_col8{
            font-weight:  bold;
        }</style><table id="T_137299cc_1209_11ed_99db_acde48001122" ><caption>Text Cleaning Word Count Comparison</caption><thead>    <tr>        <th class="col_heading level0 col0" >Company</th>        <th class="col_heading level0 col1" >Item 1A Original</th>        <th class="col_heading level0 col2" >Item 1A Shortened</th>        <th class="col_heading level0 col3" >Item 1A Stop Words & Punctuation</th>        <th class="col_heading level0 col4" >Item 7 Original</th>        <th class="col_heading level0 col5" >Item 7 Shortened</th>        <th class="col_heading level0 col6" >Item 7 Stop Words & Punctuation</th>        <th class="col_heading level0 col7" >Total Stop Words & Punctuation</th>        <th class="col_heading level0 col8" >% of Words Removed</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Pfizer Inc.</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col1" class="data row0 col1" >10668</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col2" class="data row0 col2" >6045</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col3" class="data row0 col3" >4623</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col4" class="data row0 col4" >16942</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col5" class="data row0 col5" >10640</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col6" class="data row0 col6" >6302</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col7" class="data row0 col7" >10925</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row0_col8" class="data row0 col8" >39.6%</td>
            </tr>
            <tr>
                                <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Moderna, Inc.</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col1" class="data row1 col1" >32508</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col2" class="data row1 col2" >17472</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col3" class="data row1 col3" >15036</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col4" class="data row1 col4" >8484</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col5" class="data row1 col5" >5096</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col6" class="data row1 col6" >3388</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col7" class="data row1 col7" >18424</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row1_col8" class="data row1 col8" >44.9%</td>
            </tr>
            <tr>
                                <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Johnson & Johnson</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col1" class="data row2 col1" >5972</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col2" class="data row2 col2" >3613</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col3" class="data row2 col3" >2359</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col4" class="data row2 col4" >10238</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col5" class="data row2 col5" >6703</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col6" class="data row2 col6" >3535</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col7" class="data row2 col7" >5894</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row2_col8" class="data row2 col8" >36.4%</td>
            </tr>
            <tr>
                                <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Eli Lilly and Company</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col1" class="data row3 col1" >5894</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col2" class="data row3 col2" >3353</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col3" class="data row3 col3" >2541</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col4" class="data row3 col4" >10483</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col5" class="data row3 col5" >6497</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col6" class="data row3 col6" >3986</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col7" class="data row3 col7" >6527</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row3_col8" class="data row3 col8" >39.9%</td>
            </tr>
            <tr>
                                <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col0" class="data row4 col0" >AbbVie Inc.</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col1" class="data row4 col1" >7759</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col2" class="data row4 col2" >4942</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col3" class="data row4 col3" >2817</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col4" class="data row4 col4" >9415</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col5" class="data row4 col5" >6123</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col6" class="data row4 col6" >3292</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col7" class="data row4 col7" >6109</td>
                        <td id="T_137299cc_1209_11ed_99db_acde48001122row4_col8" class="data row4 col8" >35.6%</td>
            </tr>
    </tbody></table>



## II. Comparison of Sentiment Word Frequency

### A) Create lists for sentiment words

For this analysis, I will use the Master Dictionary created by [Loughran and McDonald] (https://sraf.nd.edu/loughranmcdonald-master-dictionary/), which includes 354 positive and 2355 negative words that have frequently appeared in 10-k's. 

**Import Master Dictionary**


```python
master_dict=pd.read_csv(
            "/Users/chenmouse/Desktop/NLP/Final Project - 10K Sentiment Analysis/Loughran-McDonald_MasterDict.csv")
```

Taking a look at first 10 rows


```python
master_dict.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Word</th>
      <th>Seq_num</th>
      <th>Word Count</th>
      <th>Word Proportion</th>
      <th>Average Proportion</th>
      <th>Std Dev</th>
      <th>Doc Count</th>
      <th>Negative</th>
      <th>Positive</th>
      <th>Uncertainty</th>
      <th>Litigious</th>
      <th>Strong_Modal</th>
      <th>Weak_Modal</th>
      <th>Constraining</th>
      <th>Syllables</th>
      <th>Source</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AARDVARK</td>
      <td>1</td>
      <td>354</td>
      <td>1.550000e-08</td>
      <td>1.420000e-08</td>
      <td>3.820000e-06</td>
      <td>99</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AARDVARKS</td>
      <td>2</td>
      <td>3</td>
      <td>1.310000e-10</td>
      <td>8.650000e-12</td>
      <td>9.240000e-09</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABACI</td>
      <td>3</td>
      <td>9</td>
      <td>3.940000e-10</td>
      <td>1.170000e-10</td>
      <td>5.290000e-08</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABACK</td>
      <td>4</td>
      <td>29</td>
      <td>1.270000e-09</td>
      <td>6.650000e-10</td>
      <td>1.600000e-07</td>
      <td>28</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABACUS</td>
      <td>5</td>
      <td>8570</td>
      <td>3.750000e-07</td>
      <td>3.810000e-07</td>
      <td>3.530000e-05</td>
      <td>1108</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ABACUSES</td>
      <td>6</td>
      <td>0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ABAFT</td>
      <td>7</td>
      <td>4</td>
      <td>1.750000e-10</td>
      <td>2.300000e-11</td>
      <td>2.460000e-08</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ABALONE</td>
      <td>8</td>
      <td>142</td>
      <td>6.220000e-09</td>
      <td>4.970000e-09</td>
      <td>1.070000e-06</td>
      <td>48</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ABALONES</td>
      <td>9</td>
      <td>1</td>
      <td>4.380000e-11</td>
      <td>8.280000e-11</td>
      <td>8.850000e-08</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>12of12inf</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ABANDON</td>
      <td>10</td>
      <td>127090</td>
      <td>5.560000e-06</td>
      <td>4.700000e-06</td>
      <td>3.310000e-05</td>
      <td>66312</td>
      <td>2009</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>12of12inf</td>
    </tr>
  </tbody>
</table>
</div>



**Filter for sentiment words, which are indicated by the year added to the master dictionary under "Negative" and "Positive" columns**

List of negative words


```python
neg_words_ls=master_dict[(master_dict['Negative']!=0)]['Word'].tolist()
neg_words_ls=[i.lower() for i in neg_words_ls]
print(neg_words_ls[0:10])
```

    ['abandon', 'abandoned', 'abandoning', 'abandonment', 'abandonments', 'abandons', 'abdicated', 'abdicates', 'abdicating', 'abdication']


List of positive words


```python
pos_words_ls=master_dict[(master_dict['Positive']!=0)]['Word'].tolist()
pos_words_ls=[i.lower() for i in pos_words_ls]
print(pos_words_ls[0:10])
```

    ['able', 'abundance', 'abundant', 'acclaimed', 'accomplish', 'accomplished', 'accomplishes', 'accomplishing', 'accomplishment', 'accomplishments']



```python
len(neg_words_ls), len(pos_words_ls)
```




    (2355, 354)



**Notice that the dictionary is comprehensive of all word forms (i.e. all forms of "accomplish"), thus there is no need to lemmatize item 1A and item 7 dictionaries with no stop words.**

### B) Count number of positive and negative words in 10-k's

Count number of positive words


```python
def count_positive(dict, ticker):
    pos_count=0
    for i in range(0,len(word_tokenize(dict[ticker]))):
        if word_tokenize(dict[ticker])[i] in pos_words_ls:
            pos_count+=1
    return pos_count
```


```python
def count_negative(dict, ticker):
    neg_count=0
    for i in range(0,len(word_tokenize(dict[ticker]))):
        if word_tokenize(dict[ticker])[i] in neg_words_ls:
            neg_count+=1
    return neg_count
```

Count number of positive and negative words


```python
item1A_pos_count={}
item7_pos_count={}
item1A_neg_count={}
item7_neg_count={}
for i in range(0, len(ticker_ls)):
    item1A_pos_count[ticker_ls[i]]=count_positive(item1A_nsw_dict, ticker_ls[i])
    item7_pos_count[ticker_ls[i]]=count_positive(item7_nsw_dict, ticker_ls[i])
    item1A_neg_count[ticker_ls[i]]=count_negative(item1A_nsw_dict, ticker_ls[i])
    item7_neg_count[ticker_ls[i]]=count_negative(item7_nsw_dict, ticker_ls[i])
```


```python
item1A_pos_count, item1A_neg_count, item7_pos_count, item7_neg_count
```




    ({'pfe': 111, 'mrna': 453, 'jnj': 75, 'lly': 70, 'abbv': 86},
     {'pfe': 417, 'mrna': 1291, 'jnj': 290, 'lly': 338, 'abbv': 304},
     {'pfe': 179, 'mrna': 58, 'jnj': 116, 'lly': 92, 'abbv': 108},
     {'pfe': 226, 'mrna': 46, 'jnj': 138, 'lly': 164, 'abbv': 78})




```python
def word_count_table(item_count_pos, item_count_neg, item_nsw):
    pos=pd.Series(item_count_pos.values())
    neg=pd.Series(item_count_neg.values())
    wordcount=pd.DataFrame({'Company': company,'Total w/o Stop Words': item_nsw,
              'Positive Words': pos,
              'Positive Words Frequency': (pos/item_nsw),
              'Negative Words': neg,
              'Negative Words Frequency': (neg/item_nsw)})
    wordcount=wordcount.sort_values(by=['Negative Words Frequency'],ascending=False)
    wordcount['Positive Words Frequency']=wordcount['Positive Words Frequency'].map('{:.1%}'.format)
    wordcount['Negative Words Frequency']=wordcount['Negative Words Frequency'].map('{:.1%}'.format)
    return wordcount
```


```python
sentiment1a=word_count_table(item1A_pos_count, item1A_neg_count, item1A_nsw)
sentiment7=word_count_table(item7_pos_count, item7_neg_count, item7_nsw)
```


```python
styles = [dict(selector="caption",props=[("text-align", "center"), ("font-size", "100%"), ("color", 'black'),
                   ("font-weight", "bold")])]
```


```python
sentiment1a.style.hide_index().set_properties(subset=['Positive Words Frequency','Negative Words Frequency'], 
**{'font-weight': 'bold'}).set_caption('Item 1A Sentiment Word Count').set_table_styles(styles)
```




<style  type="text/css" >
    #T_f871dd18_0fbf_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }#T_f871dd18_0fbf_11ed_99db_acde48001122row0_col3,#T_f871dd18_0fbf_11ed_99db_acde48001122row0_col5,#T_f871dd18_0fbf_11ed_99db_acde48001122row1_col3,#T_f871dd18_0fbf_11ed_99db_acde48001122row1_col5,#T_f871dd18_0fbf_11ed_99db_acde48001122row2_col3,#T_f871dd18_0fbf_11ed_99db_acde48001122row2_col5,#T_f871dd18_0fbf_11ed_99db_acde48001122row3_col3,#T_f871dd18_0fbf_11ed_99db_acde48001122row3_col5,#T_f871dd18_0fbf_11ed_99db_acde48001122row4_col3,#T_f871dd18_0fbf_11ed_99db_acde48001122row4_col5{
            font-weight:  bold;
        }</style><table id="T_f871dd18_0fbf_11ed_99db_acde48001122" ><caption>Item 1A Sentiment Word Count</caption><thead>    <tr>        <th class="col_heading level0 col0" >Company</th>        <th class="col_heading level0 col1" >Total w/o Stop Words</th>        <th class="col_heading level0 col2" >Positive Words</th>        <th class="col_heading level0 col3" >Positive Words Frequency</th>        <th class="col_heading level0 col4" >Negative Words</th>        <th class="col_heading level0 col5" >Negative Words Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Eli Lilly and Company</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col1" class="data row0 col1" >3353</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col2" class="data row0 col2" >70</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col3" class="data row0 col3" >2.1%</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col4" class="data row0 col4" >338</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row0_col5" class="data row0 col5" >10.1%</td>
            </tr>
            <tr>
                                <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Johnson & Johnson</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col1" class="data row1 col1" >3613</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col2" class="data row1 col2" >75</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col3" class="data row1 col3" >2.1%</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col4" class="data row1 col4" >290</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row1_col5" class="data row1 col5" >8.0%</td>
            </tr>
            <tr>
                                <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Moderna, Inc.</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col1" class="data row2 col1" >17472</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col2" class="data row2 col2" >453</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col3" class="data row2 col3" >2.6%</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col4" class="data row2 col4" >1291</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row2_col5" class="data row2 col5" >7.4%</td>
            </tr>
            <tr>
                                <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Pfizer Inc.</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col1" class="data row3 col1" >6045</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col2" class="data row3 col2" >111</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col3" class="data row3 col3" >1.8%</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col4" class="data row3 col4" >417</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row3_col5" class="data row3 col5" >6.9%</td>
            </tr>
            <tr>
                                <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col0" class="data row4 col0" >AbbVie Inc.</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col1" class="data row4 col1" >4942</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col2" class="data row4 col2" >86</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col3" class="data row4 col3" >1.7%</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col4" class="data row4 col4" >304</td>
                        <td id="T_f871dd18_0fbf_11ed_99db_acde48001122row4_col5" class="data row4 col5" >6.2%</td>
            </tr>
    </tbody></table>




```python
sentiment7.style.hide_index().set_properties(subset=['Positive Words Frequency','Negative Words Frequency'], 
**{'font-weight': 'bold'}).set_caption('Item 7 Sentiment Word Count').set_table_styles(styles)
```




<style  type="text/css" >
    #T_f87368ea_0fbf_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }#T_f87368ea_0fbf_11ed_99db_acde48001122row0_col3,#T_f87368ea_0fbf_11ed_99db_acde48001122row0_col5,#T_f87368ea_0fbf_11ed_99db_acde48001122row1_col3,#T_f87368ea_0fbf_11ed_99db_acde48001122row1_col5,#T_f87368ea_0fbf_11ed_99db_acde48001122row2_col3,#T_f87368ea_0fbf_11ed_99db_acde48001122row2_col5,#T_f87368ea_0fbf_11ed_99db_acde48001122row3_col3,#T_f87368ea_0fbf_11ed_99db_acde48001122row3_col5,#T_f87368ea_0fbf_11ed_99db_acde48001122row4_col3,#T_f87368ea_0fbf_11ed_99db_acde48001122row4_col5{
            font-weight:  bold;
        }</style><table id="T_f87368ea_0fbf_11ed_99db_acde48001122" ><caption>Item 7 Sentiment Word Count</caption><thead>    <tr>        <th class="col_heading level0 col0" >Company</th>        <th class="col_heading level0 col1" >Total w/o Stop Words</th>        <th class="col_heading level0 col2" >Positive Words</th>        <th class="col_heading level0 col3" >Positive Words Frequency</th>        <th class="col_heading level0 col4" >Negative Words</th>        <th class="col_heading level0 col5" >Negative Words Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Eli Lilly and Company</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col1" class="data row0 col1" >6497</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col2" class="data row0 col2" >92</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col3" class="data row0 col3" >1.4%</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col4" class="data row0 col4" >164</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row0_col5" class="data row0 col5" >2.5%</td>
            </tr>
            <tr>
                                <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Pfizer Inc.</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col1" class="data row1 col1" >10640</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col2" class="data row1 col2" >179</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col3" class="data row1 col3" >1.7%</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col4" class="data row1 col4" >226</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row1_col5" class="data row1 col5" >2.1%</td>
            </tr>
            <tr>
                                <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Johnson & Johnson</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col1" class="data row2 col1" >6703</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col2" class="data row2 col2" >116</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col3" class="data row2 col3" >1.7%</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col4" class="data row2 col4" >138</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row2_col5" class="data row2 col5" >2.1%</td>
            </tr>
            <tr>
                                <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col0" class="data row3 col0" >AbbVie Inc.</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col1" class="data row3 col1" >6123</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col2" class="data row3 col2" >108</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col3" class="data row3 col3" >1.8%</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col4" class="data row3 col4" >78</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row3_col5" class="data row3 col5" >1.3%</td>
            </tr>
            <tr>
                                <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Moderna, Inc.</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col1" class="data row4 col1" >5096</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col2" class="data row4 col2" >58</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col3" class="data row4 col3" >1.1%</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col4" class="data row4 col4" >46</td>
                        <td id="T_f87368ea_0fbf_11ed_99db_acde48001122row4_col5" class="data row4 col5" >0.9%</td>
            </tr>
    </tbody></table>




```python
sentiment_total=sentiment1a.add(sentiment7)
sentiment_total['Company']=company
sentiment_total['Positive Words Frequency']=(sentiment_total['Positive Words']
                                             /sentiment_total['Total w/o Stop Words']).map('{:.1%}'.format)
sentiment_total['Negative Words Frequency']=(sentiment_total['Negative Words']
                                             /sentiment_total['Total w/o Stop Words']).map('{:.1%}'.format)
```


```python
sentiment_total.sort_values(by=['Negative Words Frequency'], ascending=False).style.hide_index().set_properties(
                            subset=['Positive Words Frequency','Negative Words Frequency'], 
**{'font-weight': 'bold'}).set_caption('Total Sentiment Word Count').set_table_styles(styles)
```




<style  type="text/css" >
    #T_f87632c8_0fbf_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }#T_f87632c8_0fbf_11ed_99db_acde48001122row0_col3,#T_f87632c8_0fbf_11ed_99db_acde48001122row0_col5,#T_f87632c8_0fbf_11ed_99db_acde48001122row1_col3,#T_f87632c8_0fbf_11ed_99db_acde48001122row1_col5,#T_f87632c8_0fbf_11ed_99db_acde48001122row2_col3,#T_f87632c8_0fbf_11ed_99db_acde48001122row2_col5,#T_f87632c8_0fbf_11ed_99db_acde48001122row3_col3,#T_f87632c8_0fbf_11ed_99db_acde48001122row3_col5,#T_f87632c8_0fbf_11ed_99db_acde48001122row4_col3,#T_f87632c8_0fbf_11ed_99db_acde48001122row4_col5{
            font-weight:  bold;
        }</style><table id="T_f87632c8_0fbf_11ed_99db_acde48001122" ><caption>Total Sentiment Word Count</caption><thead>    <tr>        <th class="col_heading level0 col0" >Company</th>        <th class="col_heading level0 col1" >Total w/o Stop Words</th>        <th class="col_heading level0 col2" >Positive Words</th>        <th class="col_heading level0 col3" >Positive Words Frequency</th>        <th class="col_heading level0 col4" >Negative Words</th>        <th class="col_heading level0 col5" >Negative Words Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Moderna, Inc.</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col1" class="data row0 col1" >22568</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col2" class="data row0 col2" >511</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col3" class="data row0 col3" >2.3%</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col4" class="data row0 col4" >1337</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row0_col5" class="data row0 col5" >5.9%</td>
            </tr>
            <tr>
                                <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Eli Lilly and Company</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col1" class="data row1 col1" >9850</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col2" class="data row1 col2" >162</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col3" class="data row1 col3" >1.6%</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col4" class="data row1 col4" >502</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row1_col5" class="data row1 col5" >5.1%</td>
            </tr>
            <tr>
                                <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Johnson & Johnson</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col1" class="data row2 col1" >10316</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col2" class="data row2 col2" >191</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col3" class="data row2 col3" >1.9%</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col4" class="data row2 col4" >428</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row2_col5" class="data row2 col5" >4.1%</td>
            </tr>
            <tr>
                                <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Pfizer Inc.</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col1" class="data row3 col1" >16685</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col2" class="data row3 col2" >290</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col3" class="data row3 col3" >1.7%</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col4" class="data row3 col4" >643</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row3_col5" class="data row3 col5" >3.9%</td>
            </tr>
            <tr>
                                <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col0" class="data row4 col0" >AbbVie Inc.</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col1" class="data row4 col1" >11065</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col2" class="data row4 col2" >194</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col3" class="data row4 col3" >1.8%</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col4" class="data row4 col4" >382</td>
                        <td id="T_f87632c8_0fbf_11ed_99db_acde48001122row4_col5" class="data row4 col5" >3.5%</td>
            </tr>
    </tbody></table>



### C) Conclusion

With the range of Positive Words Frequency being much narrower than that of Negative Words Frequency among the five companies, Negative Words Frequency serves as a differentiator in the tones of the 10-k financial reports. 

Overall, Moderna, Inc. ranks highest in frequency of negative words, followed by Eli Lilly and Company, Johnson & Johnson, Pfizer Inc. and AbbVie Inc. However, for the individual items, Eli Lilly and Company ranks highest instead. It appears Moderna's high overall ranking is mostly attributable to high negative words frequency in item 1A.

Zooming in on item 7, which discusses the company's annual performance in greater detail than item 1A, Eli Lilly and Company, Pfizer Inc. and Johnson and Johnson's reports are worth a more in-depth read as their tones indicate more negative sentiments than the rest. 

We will shed more light on this analysis in section II, higlighting the sentiment words that appear most frequently and the sentences they belong to.

## II. Identification of Most Frequent Sentiment Words

**Combine positive and negative words into one sentiment word list**


```python
sent_word_ls=neg_words_ls+pos_words_ls
```

**Retrieve 10 most frequently appearing sentiment words in item 1A and 7 as dictionaries**


```python
def top_10_sent_words(item_dict, ticker):
    sent_word_dict={}
    for i in range(0, len(sent_word_ls)):
        sent_word_dict[sent_word_ls[i]]=word_tokenize(item_dict[ticker]).count(sent_word_ls[i])
    sent_word_dict={k: v for k, v in sorted(sent_word_dict.items(), key=lambda item: item[1])}
    top10ls=list(sent_word_dict)[-10:]
    top10={}
    for i in range(0, len(top10ls)):
        top10[top10ls[i]]=sent_word_dict[top10ls[i]]
    return top10
```


```python
top10pfe_1a=top_10_sent_words(item1A_nsw_dict, 'pfe')
top10mrna_1a=top_10_sent_words(item1A_nsw_dict, 'mrna')
top10jnj_1a=top_10_sent_words(item1A_nsw_dict, 'jnj')
top10lly_1a=top_10_sent_words(item1A_nsw_dict, 'lly')
top10abbv_1a=top_10_sent_words(item1A_nsw_dict, 'abbv')
top10pfe_7=top_10_sent_words(item7_nsw_dict, 'pfe')
top10mrna_7=top_10_sent_words(item7_nsw_dict, 'mrna')
top10jnj_7=top_10_sent_words(item7_nsw_dict, 'jnj')
top10lly_7=top_10_sent_words(item7_nsw_dict, 'lly')
top10abbv_7=top_10_sent_words(item7_nsw_dict, 'abbv')
```

**Compile all dictionaries into a Dataframe**


```python
top101adict_ls=[top10pfe_1a, top10mrna_1a, top10jnj_1a, top10lly_1a, top10abbv_1a]
top107dict_ls=[top10pfe_7, top10mrna_7, top10jnj_7, top10lly_7, top10abbv_7]
```


```python
def top_10_dict(company_name, dict):
    name_sr=pd.Series([company_name]*10)
    top10df=pd.DataFrame(dict.items(), columns=['Sentiment Word', 'Frequency'])
    top10df['Sentiment Word']=top10df['Sentiment Word'].str.capitalize()
    top10df=top10df.sort_values(['Frequency'], ascending=False).reset_index(drop=True)
    return top10df
```


```python
def top10_df(item_dict_ls,item_str):
    df1 = top_10_dict(company_ls[0], item_dict_ls[0])
    df2 = top_10_dict(company_ls[1], item_dict_ls[1])
    df3 = top_10_dict(company_ls[2], item_dict_ls[2])
    df4 = top_10_dict(company_ls[3], item_dict_ls[3])
    df5 = top_10_dict(company_ls[4], item_dict_ls[4])
    df1_style = df1.style.hide_index().set_table_attributes("style='display:inline; margin-right:20px;'")
                .set_caption(item_str+str(company_ls[0])).set_table_styles(styles)
    df2_style = df2.style.hide_index().set_table_attributes("style='display:inline; margin-right:20px;'")
                .set_caption(item_str+str(company_ls[1])).set_table_styles(styles)
    df3_style = df3.style.hide_index().set_table_attributes("style='display:inline; margin-right:20px;'")
                .set_caption(item_str+str(company_ls[2])).set_table_styles(styles)
    df4_style = df4.style.hide_index().set_table_attributes("style='display:inline; margin-right:20px;'")
                .set_caption(item_str+str(company_ls[3])).set_table_styles(styles)
    df5_style = df5.style.hide_index().set_table_attributes("style='display:inline'")
                .set_caption(item_str+ str(company_ls[4])).set_table_styles(styles)
    return display_html(df1_style._repr_html_() + df2_style._repr_html_() + df3_style._repr_html_() 
                 + df4_style._repr_html_() + df5_style._repr_html_(), raw=True)
```

**Item 1A Most Frequent Sentiment Words**


```python
top10_df(top101adict_ls,"Item 1A - ")
```


<style  type="text/css" >
    #T_bce35e82_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce35e82_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 1A - Pfizer Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Challenges</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >26</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Adversely</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >20</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Claims</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >18</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Litigation</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Adverse</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Disruptions</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Delays</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Fail</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Loss</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Able</td>
                        <td id="T_bce35e82_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >10</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce39b18_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce39b18_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 1A - Moderna, Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Adversely</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >71</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Collaborators</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >55</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Adverse</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >47</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Delays</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >45</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Claims</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >43</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Fail</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >40</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Delay</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >36</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Failure</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >34</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Able</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >32</td>
            </tr>
            <tr>
                                <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Litigation</td>
                        <td id="T_bce39b18_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >28</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce3df60_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce3df60_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 1A - Johnson & Johnson</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Adversely</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >23</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Litigation</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Delays</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Challenges</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Negatively</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Damage</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Investigations</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Loss</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Successful</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Effective</td>
                        <td id="T_bce3df60_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >6</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce42b82_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce42b82_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 1A - Eli Lilly and Company</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Adversely</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >21</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Litigation</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >13</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Failure</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Unauthorized</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Claims</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Successful</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Disruption</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Failures</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Investigations</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Loss</td>
                        <td id="T_bce42b82_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >7</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce47ca4_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce47ca4_0fc1_11ed_99db_acde48001122" style='display:inline'><caption>Item 1A - AbbVie Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Adverse</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >27</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Adversely</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >23</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Failure</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >16</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Successful</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Impairment</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Profitability</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Loss</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Negatively</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Restated</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Successfully</td>
                        <td id="T_bce47ca4_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >7</td>
            </tr>
    </tbody></table>



```python
top10_df(top107dict_ls,"Item 7 - ")
```


<style  type="text/css" >
    #T_bce7aeba_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce7aeba_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 7 - Pfizer Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Benefit</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >25</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Collaboration</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >23</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Decline</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >19</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Discontinued</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >19</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Gains</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >19</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Impairment</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >18</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Restructuring</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >15</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Losses</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Greater</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Unfavorable</td>
                        <td id="T_bce7aeba_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >9</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce7e56a_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce7e56a_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 7 - Moderna, Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Loss</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Collaboration</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Alliances</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >4</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Benefit</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >4</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Enable</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >4</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Termination</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Advances</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Collaborators</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Effective</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Progress</td>
                        <td id="T_bce7e56a_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >3</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce81530_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce81530_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 7 - Johnson & Johnson</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Loss</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >16</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Gains</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >13</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Losses</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Achieved</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Litigation</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Benefit</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Positive</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Doubtful</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Negative</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce81530_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Strong</td>
                        <td id="T_bce81530_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >7</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce85b08_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce85b08_0fc1_11ed_99db_acde48001122" style='display:inline; margin-right:20px;'><caption>Item 7 - Eli Lilly and Company</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Impairment</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >17</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Benefit</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >17</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Loss</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Litigation</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Adversely</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Failure</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Exclusivity</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Severe</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Favorable</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Gains</td>
                        <td id="T_bce85b08_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >6</td>
            </tr>
    </tbody></table><style  type="text/css" >
    #T_bce89b40_0fc1_11ed_99db_acde48001122 caption {
          text-align: center;
          font-size: 100%;
          color: black;
          font-weight: bold;
    }</style><table id="T_bce89b40_0fc1_11ed_99db_acde48001122" style='display:inline'><caption>Item 7 - AbbVie Inc.</caption><thead>    <tr>        <th class="col_heading level0 col0" >Sentiment Word</th>        <th class="col_heading level0 col1" >Frequency</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row0_col0" class="data row0 col0" >Benefit</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row0_col1" class="data row0 col1" >18</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row1_col0" class="data row1 col0" >Collaboration</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row1_col1" class="data row1 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row2_col0" class="data row2 col0" >Strong</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row2_col1" class="data row2 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row3_col0" class="data row3 col0" >Loss</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row3_col1" class="data row3 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row4_col0" class="data row4 col0" >Severe</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row4_col1" class="data row4 col1" >9</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row5_col0" class="data row5 col0" >Impairment</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row5_col1" class="data row5 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row6_col0" class="data row6 col0" >Favorable</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row6_col1" class="data row6 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row7_col0" class="data row7 col0" >Inadequate</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row7_col1" class="data row7 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row8_col0" class="data row8 col0" >Favorably</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row8_col1" class="data row8 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row9_col0" class="data row9 col0" >Effective</td>
                        <td id="T_bce89b40_0fc1_11ed_99db_acde48001122row9_col1" class="data row9 col1" >5</td>
            </tr>
    </tbody></table>


You may notice one or two of the top 10 most frequent sentiment words being of the same root word (i.e. "adverse" and "adversely"). This is due to lemmatization being skipped to save significant time on running this notebook. Regardless, the output still produces top 8-9 words. 

**Conclusion**

Item 1A tends to include more negative words because it discusses risk factors. This is consistent with the five companies. </mark> Item 7 shows a more diverse range of sentiments, with a more even split between positive and negative words. In conjunction with the quantitative comparison of sentiment words frequency in section I, searching for the most frequent sentiment words can lead to informative insight, with the following examples: 
    
**AbbVie Inc., Item 1A** <br>
"Successful(ly)", "Adversely"<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The <mark style="background-color: yellow">**successful**</mark> discovery, development, manufacturing and sale of biologics is a long, expensive and uncertain process. There are unique risks and uncertainties with biologics. For example, access to and supply of necessary biological materials, such as cell lines, may be limited and governmental regulations restrict access to and regulate the transport and use of such materials. In addition, the development, manufacturing and sale of biologics is subject to regulations that are often more complex and extensive than the regulations applicable to other pharmaceutical products...Biologics are also frequently costly to manufacture because production inputs are derived from living animal or plant material, and some biologics cannot be made synthetically. Failure to <mark style="background-color: yellow">**successfully**</mark> discover, develop, manufacture and sell biologics—including Humira—could <mark style="background-color: yellow">**adversely**</mark> impact AbbVie's business and results of operations.  

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


```python

```
