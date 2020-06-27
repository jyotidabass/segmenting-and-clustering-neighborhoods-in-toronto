# segmenting-and-clustering-neighborhoods-in-toronto
Segmenting and Clustering Neighborhoods in Toronto
In this assignment, you will be required to explore, segment, and cluster the neighborhoods in the city of Toronto. However, unlike New York, the neighborhood data is not readily available on the internet. What is interesting about the field of data science is that each project can be challenging in its unique way, so you need to learn to be agile and refine the skill to learn new libraries and tools quickly depending on the project.

For the Toronto neighborhood data, a Wikipedia page exists that has all the information we need to explore and cluster the neighborhoods in Toronto. You will be required to scrape the Wikipedia page and wrangle the data, clean it, and then read it into a pandas dataframe so that it is in a structured format like the New York dataset.

Once the data is in a structured format, you can replicate the analysis that we did to the New York City dataset to explore and cluster the neighborhoods in the city of Toronto.

Step 1 - Scrap Wikipedia for data using BeautifulSoup
Use the Notebook to build the code to scrape the following Wikipedia page, https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M, in order to obtain the data that is in the table of postal codes and to transform the data into a pandas dataframe like the one shown below:

To create the dataframe:
The dataframe will consist of three columns: PostalCode, Borough, and Neighborhood.
Only process the cells that have an assigned borough. Ignore cells with a borough that is Not assigned.
More than one neighborhood can exist in one postal code area. For example, in the table on the Wikipedia page, you will notice that M5A is listed twice and has two neighborhoods: Harbourfront and Regent Park. These two rows will be combined into one row with the neighborhoods separated with a comma as shown in row 11 in the above table.
If a cell has a borough but a Not assigned neighborhood, then the neighborhood will be the same as the borough. So for the 9th cell in the table on the Wikipedia page, the value of the Borough and the Neighborhood columns will be Queen's Park.
Clean your Notebook and add Markdown cells to explain your work and any assumptions you are making.
In the last cell of your notebook, use the .shape method to print the number of rows of your dataframe.
Note: There are different website scraping libraries and packages in Python. One of the most common packages is BeautifulSoup. Here is the package's main documentation page: http://beautiful-soup-4.readthedocs.io/en/latest/


In [1]:
#install Beautiful Soup and requests for Web Scaping
!pip install BeautifulSoup4
!pip install requests


Requirement already satisfied: BeautifulSoup4 in /home/jupyterlab/conda/lib/python3.6/site-packages (4.7.1)
Requirement already satisfied: soupsieve>=1.2 in /home/jupyterlab/conda/lib/python3.6/site-packages (from BeautifulSoup4) (1.7.1)
Requirement already satisfied: requests in /home/jupyterlab/conda/lib/python3.6/site-packages (2.21.0)
Requirement already satisfied: certifi>=2017.4.17 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests) (2018.11.29)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests) (3.0.4)
Requirement already satisfied: urllib3<1.25,>=1.21.1 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests) (1.24.1)
Requirement already satisfied: idna<2.9,>=2.5 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests) (2.8)
Step 1A: Get Data
Get HTML from wikipedia
Use BeautifySoup to parse html data
Store parsed data into Pandas DataFrame

In [7]:
#imports
from bs4 import BeautifulSoup
import requests
import pandas as pd
import numpy as np


#get html from wiki page and create soup object
source = requests.get("https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M")
soup = BeautifulSoup(source.text, 'lxml')

#using soup object, iterate the .wikitable to get the data from the HTML page and store it into a list
data = []
columns = []
table = soup.find(class_='wikitable')
for index, tr in enumerate(table.find_all('tr')):
    section = []
    for td in tr.find_all(['th','td']):
        section.append(td.text.rstrip())
    
    #First row of data is the header
    if (index == 0):
        columns = section
    else:
        data.append(section)


#convert list into Pandas DataFrame
canada_df = pd.DataFrame(data = data,columns = columns)
canada_df.head()

Out[7]:
Postcode	Borough	Neighbourhood
0	M1A	Not assigned	Not assigned
1	M2A	Not assigned	Not assigned
2	M3A	North York	Parkwoods
3	M4A	North York	Victoria Village
4	M5A	Downtown Toronto	Harbourfront

Step 1B: Data Cleanup
Remove Boroughs that are 'Not assigned'
More than one neighborhood can exist in one postal code area, combined these into one row with the neighborhoods separated with a comma
If a cell has a borough but a Not assigned neighborhood, then the neighborhood will be the same as the borough

In [9]:
#Remove Boroughs that are 'Not assigned'
canada_df = canada_df[canada_df['Borough'] != 'Not assigned']
canada_df.head()

Out[9]:
Postcode	Borough	Neighbourhood
2	M3A	North York	Parkwoods
3	M4A	North York	Victoria Village
4	M5A	Downtown Toronto	Harbourfront
5	M5A	Downtown Toronto	Regent Park
6	M6A	North York	Lawrence Heights

In [10]:
# More than one neighborhood can exist in one postal code area, combined these into one row with the neighborhoods separated with a comma
canada_df["Neighbourhood"] = canada_df.groupby("Postcode")["Neighbourhood"].transform(lambda neigh: ', '.join(neigh))

#remove duplicates
canada_df = canada_df.drop_duplicates()

#update index to be postcode if it isn't already
if(canada_df.index.name != 'Postcode'):
    canada_df = canada_df.set_index('Postcode')
    
canada_df.head()

/home/jupyterlab/conda/lib/python3.6/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
  
Out[10]:
Borough	Neighbourhood
Postcode		
M3A	North York	Parkwoods
M4A	North York	Victoria Village
M5A	Downtown Toronto	Harbourfront, Regent Park
M6A	North York	Lawrence Heights, Lawrence Manor
M7A	Queen's Park	Not assigned
In [11]:
# If a cell has a borough but a Not assigned neighborhood, then the neighborhood will be the same as the borough
canada_df['Neighbourhood'].replace("Not assigned", canada_df["Borough"],inplace=True)
canada_df.head()

Out[11]:
Borough	Neighbourhood
Postcode		
M3A	North York	Parkwoods
M4A	North York	Victoria Village
M5A	Downtown Toronto	Harbourfront, Regent Park
M6A	North York	Lawrence Heights, Lawrence Manor
M7A	Queen's Park	Queen's Park
Step 1C - Output
In the last cell of your notebook, use the .shape method to print the number of rows of your dataframe.

In [12]:
canada_df.shape
Out[12]:
(103, 2)
