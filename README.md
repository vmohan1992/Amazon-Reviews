
# Amazon Reviews & Ratings Analysis

**Data source:** The dataset used for the analysis contains customer product reviews and ratings and metadata from Amazon, spanning May 1996-July 2014. The data was sourced from “Amazon product data” by Julian McAuley, UCSD - http://jmcauley.ucsd.edu/data/amazon/links.html

**Scope of Analysis:** For the analysis, two major product categories *“Clothing, Shoes & Jewellery”* and *“Electronics”* were used. The product reviews and metadata for these two categories with more than 5 reviews, spanning from May 1996-July 2014, were analysed to identify key insights.

**Data Preparation:** The data was available in the CSV/json format. To improve the data quality and help in the downstream data analysis, the data had to be cleaned and a few additional attributes were to be added to the data. This task was accomplished using python. The code for the same is available [here](https://github.com/vmohan1992/Amazon-Reviews/blob/73c66ded234c97b513d76c49deca64f79efc3342/Takeaway.ipynb).

**Data Analysis and Visualization:** Following the data preparation, the data was analysed using Tableau Desktop. The data was connected as a CSV and then subsequent analysis was done visually to create an insightful Tableau Dashboard. The dashboard can be accessed from here.

## Prerequisites

 1. Git
 1. An IDE to run python code ( e.g. Jupyter Notebook)

## Data

The data cane be downloaded from  http://jmcauley.ucsd.edu/data/amazon/links.html

## Implementation

1. Download and save the data in the working directory
1. Clone the [repository](https://github.com/vmohan1992/Amazon-Reviews.git) using Git 
1. Open the .ipynb filte in the IDE
1. Run the code

## Code
```
# Import all necessary packages
import pandas as pd
import numpy as np
import gzip
```
```
# Get the current working directory
import os
path = os.getcwd()
print(path)
```
```
#Read the csv files to dataframes.
#dfm - metadata for Clothing, Shoes & Jewellery
#dfc - reviews for Clothing, Shoes & Jewellery
#dfe - reviews for Electronics

dfm=pd.read_csv("metadata_category_clothing_shoes_and_jewelry_only.csv",index_col=0)
dfc= pd.read_csv("reviews_Clothing_Shoes_and_Jewelry_5.csv",index_col=0)
dfe=pd.read_csv("reviews_Electronics_5.csv",index_col=0)

```
```
#The metadata for "Electronics" is available in the json format.Code provided via http://jmcauley.ucsd.edu/data/amazon/ defines a function to parse the json file to DataFrame.
#dfm2 - Metadata for Electronics

def parse_gz(path):
    g = gzip.open(path, 'rb')
    for l in g:
        yield eval(l)

def convert_to_DF(path):a
    i = 0
    df = {}
    for d in parse_gz(path):
        df[i] = d
        i += 1
    return pd.DataFrame.from_dict(df, orient='index')
```
```
dfm2 = convert_to_DF('meta_Electronics.json.gz')
dfm2.rename(columns={'salesRank': 'salesrank'},inplace=True) 
```
```
# Convert all the columns of the metadata to string type
dfm= dfm.astype(str)
dfm2= dfm2.astype(str)
```
```
#Concatenate the metadata for clothing and electronics create one dataframe
meta=pd.concat([dfm,dfm2])
```
```
# Create a field for Category and Sales Rank by splitting the "Salesrank" column

meta[["Category","Sales_Rank"]] = pd.DataFrame(meta.salesrank.str.split(':',1,expand=True))
meta['Category']= meta['Category'].str.replace('{','',regex=True)
meta['Category']= meta['Category'].str.replace('}','',regex=True)
meta['Category']= meta['Category'].str.replace('\'','',regex=True)
meta['Sales_Rank']= meta['Sales_Rank'].str.replace('}','',regex=True)
```
```
# Add a column for Reporting Category for dfc and dfe before concatinating to idetify the product category

dfc['Reporting_category'] = pd.Series(["Clothing, Shoes & Jewellery" for x in range(len(dfc.index))])
dfe['Reporting_category'] = pd.Series(["Electronics" for x in range(len(dfe.index))])
```
```
#Concatenate dfc and dfe to create one dataframe
df=pd.concat([dfc,dfe])
```
```
# Split the 'helful' column to extract "helpful votes" & "Total Votes"

df['helpful']= df['helpful'].str.replace('[','',regex=True)
df['helpful']= df['helpful'].str.replace(']','',regex=True)
df[["helpful_votes","total_votes"]] = pd.DataFrame(df.helpful.str.split(',',1,expand=True))
df["reviewTime"] = pd.to_datetime(df["reviewTime"])
```
```
df.total_votes=df.total_votes.astype(int)
```
```
df_mean=df.groupby(['asin','reviewTime'])['overall'].mean()
```
```
# Create a new column "quality" to categorize ratings. 
# "Positive" for  4-5 star ratings,"Neutral" for 3 star ratings & "bad" for < 3 ratings 

df["quality"]=df.loc[:,"overall"].apply(lambda x:"positive" if x >= 4 else ("neutral" if x==3 else "bad"))
```
```
# Create a new column "vote range" to categorize number of votes for the reviews 

df["vote_range"]=df.loc[:,"total_votes"].apply(lambda x:"0 Votes" if x == 0 else ("1-5 Votes" if x<=5 else( "5-10 votes" if  x<=10 else ( "10-50 Votes" if x<=50 else( "50-100 Votes" if x<=100 else "Over 100 Votes")) ) ))
```
```
for i, each in enumerate(dfc.overall.value_counts()):
    print(f"Percentgae of {dfc.overall.value_counts().index[i]} starts : {(each*100/len(dfc.overall)):.2f}%")
```
```
for i, each in enumerate(dfe.overall.value_counts()):
    print(f"Percentgae of {dfe.overall.value_counts().index[i]} starts : {(each*100/len(dfe.overall)):.2f}%")
```
```
meta.head()
```
```
df.head()
```
```
# Write metadata and reviews into a csv for subsequent analysis in Tableau

meta.to_csv(r'C:\\Users\\vmoha\\documents\\Python Data\\CSV\\metadata.csv', index = False)
df.to_csv(r'C:\\Users\\vmoha\\documents\\Python Data\\CSV\\total_reviews.csv', index = False)
```

    










