# Data Acquisition and Cleaning

## Definitions:
A quarter is a specific three month period, Q1 is January through March, Q2 is April through June, 
Q3 is July through September, Q4 is October through December.
A recession is defined as starting with two consecutive quarters of GDP decline, and ending with 
two consecutive quarters of GDP growth.
A recession bottom is the quarter within a recession which had the lowest GDP.
A university town is a city which has a high percentage of university students compared to the 
total population of the city.

## Hypothesis: 
University towns have their mean housing prices less effected by recessions. 
Run a t-test to compare the ratio of the mean price of houses in university towns the quarter before 
the recession starts compared to the recession bottom. (price_ratio=quarter_before_recession/recession_bottom)

## Data: 
From the Zillow research data site there is housing data for the United States. 
In particular the datafile for all homes at a city level, City_Zhvi_AllHomes.csv, has median home sale prices 
at a fine grained level.
From the Wikipedia page on college towns is a list of university towns in the United States which has been 
copy and pasted into the file university_towns.txt.
From Bureau of Economic Analysis, US Department of Commerce, the GDP over time of the United States in 
current dollars (the chained value in 2009 dollars), in quarterly intervals, in the file gdplev.xls. 
GDP data used are from the first quarter of 2000 onward.

```
import pandas as pd
import numpy as np
import re
from scipy.stats import ttest_ind
```

Use this dictionary to map state names to two letter acronyms

```
states = {'OH': 'Ohio', 'KY': 'Kentucky', 'AS': 'American Samoa', 'NV': 'Nevada', 'WY': 'Wyoming', 
          'NA': 'National', 'AL': 'Alabama', 'MD': 'Maryland', 'AK': 'Alaska', 'UT': 'Utah', 'OR': 'Oregon', 
          'MT': 'Montana', 'IL': 'Illinois', 'TN': 'Tennessee', 'DC': 'District of Columbia', 'VT': 'Vermont', 
          'ID': 'Idaho', 'AR': 'Arkansas', 'ME': 'Maine', 'WA': 'Washington', 'HI': 'Hawaii', 'WI': 'Wisconsin', 
          'MI': 'Michigan', 'IN': 'Indiana', 'NJ': 'New Jersey', 'AZ': 'Arizona', 'GU': 'Guam', 'MS': 'Mississippi', 
          'PR': 'Puerto Rico', 'NC': 'North Carolina', 'TX': 'Texas', 'SD': 'South Dakota', 
          'MP': 'Northern Mariana Islands', 'IA': 'Iowa', 'MO': 'Missouri', 'CT': 'Connecticut', 
          'WV': 'West Virginia', 'SC': 'South Carolina', 'LA': 'Louisiana', 'KS': 'Kansas', 'NY': 'New York', 
          'NE': 'Nebraska', 'OK': 'Oklahoma', 'FL': 'Florida', 'CA': 'California', 'CO': 'Colorado', 
          'PA': 'Pennsylvania', 'DE': 'Delaware', 'NM': 'New Mexico', 'RI': 'Rhode Island', 'MN': 'Minnesota', 
          'VI': 'Virgin Islands', 'NH': 'New Hampshire', 'MA': 'Massachusetts', 'GA': 'Georgia', 'ND': 'North Dakota', 
          'VA': 'Virginia'}
```

Convert the university_towns.txt list to a DataFrame of towns and the states they are in. 
The format of the DataFrame is: DataFrame( [ ["Michigan", "Ann Arbor"], ["Michigan", "Yipsilanti"] ], 
columns=["State", "RegionName"]  )

The following cleaning has been done:

1. For "State", removing characters from "[" to the end.
2. For "RegionName", when applicable, removing every character from either "[" or " (" or ":" to the end.
3. Note: certain rows in the data represent universities and not towns. However, those rows will get dropped
   after merging with housing price data. 


```
def get_list_of_university_towns():
    university_towns = pd.read_table('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/university_towns.txt', header = None)
    university_towns.columns = ['RegionName']
    university_towns['State'] = np.nan
    for town in university_towns['RegionName']:
        if re.search('(\[edit\])', town): 
            university_towns.State[university_towns.RegionName == town] = town
    university_towns['State'] = university_towns['State'].fillna(method = 'ffill')
    university_towns = university_towns[university_towns.RegionName != university_towns.State]
    university_towns = university_towns.reset_index(drop = True)
    for town in university_towns.RegionName:
        town_edited = re.sub('([\(\[:].*)', '', town).rstrip()
        #town_edited = re.sub('(\[.*)', '', town_edited).rstrip()
        university_towns.RegionName[university_towns.RegionName == town] = town_edited
    for state in university_towns.State:
        state_edited = re.sub('(\[.*)', '', state).rstrip()
        university_towns.State[university_towns.State == state] = state_edited
        names = ['State', 'RegionName']
        university_towns = university_towns[names]
    return university_towns
#a = get_list_of_university_towns()
#print(a)
```


Return the year and quarter of the recession start time as a string value in a format such as 2005Q3

```
def get_recession_start():
    gdp = pd.read_excel('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/gdplev.xlsx', skiprows = 5)
    gdp = gdp[['Unnamed: 4', 'GDP in billions of chained 2009 dollars.1']]
    gdp.rename(columns = {'Unnamed: 4': 'Quarter', 'GDP in billions of chained 2009 dollars.1': 'GDP'}, inplace = True)
    i = gdp.index[gdp.Quarter == '2000Q1'][0]
    gdp = gdp[(gdp.index >= i)]
    gdp = gdp.reset_index(drop = True)
    gdp['Lagged GDP'] = np.nan
    gdp['Lagged GDP'][1:] = gdp.GDP[0:-1]
    gdp['Change in GDP'] = gdp.GDP - gdp['Lagged GDP']
    gdp['Lagged Change in GDP'] = np.nan
    gdp['Lagged Change in GDP'][1:] = gdp['Change in GDP'][0:-1]
    gdp['Lead Change in GDP'] = np.nan
    gdp['Lead Change in GDP'][0:-1] = gdp['Change in GDP'][1:]
    gdp['Recession Start Dummy'] = 0
    gdp['Recession Start Dummy'][(gdp['Change in GDP'] < 0) & (gdp['Lead Change in GDP'] < 0)] = 1
    recession_start = gdp[gdp['Recession Start Dummy'] == 1]
    recession_start = recession_start.reset_index(drop = True)
    recession_start = recession_start['Quarter'].iloc[0]
    return recession_start
#a = get_recession_start()
#print(a)
```

Return the year and quarter of the recession end time as a string value in a format such as 2005Q3

```
def get_recession_end():
    gdp = pd.read_excel('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/gdplev.xlsx', skiprows = 5)
    gdp = gdp[['Unnamed: 4', 'GDP in billions of chained 2009 dollars.1']]
    gdp.rename(columns = {'Unnamed: 4': 'Quarter', 'GDP in billions of chained 2009 dollars.1': 'GDP'}, inplace = True)
    i = gdp.index[gdp.Quarter == '2000Q1'][0]
    gdp = gdp[(gdp.index >= i)]
    gdp = gdp.reset_index(drop = True)
    gdp['Lagged GDP'] = np.nan
    gdp['Lagged GDP'][1:] = gdp.GDP[0:-1]
    gdp['Change in GDP'] = gdp.GDP - gdp['Lagged GDP']
    gdp['Lagged Change in GDP'] = np.nan
    gdp['Lagged Change in GDP'][1:] = gdp['Change in GDP'][0:-1]
    gdp['Lead Change in GDP'] = np.nan
    gdp['Lead Change in GDP'][0:-1] = gdp['Change in GDP'][1:]
    gdp['Recession Start Dummy'] = 0
    gdp['Recession Start Dummy'][(gdp['Change in GDP'] < 0) & (gdp['Lead Change in GDP'] < 0)] = 1
    recession_start = gdp[gdp['Recession Start Dummy'] == 1]
    recession_start = recession_start.reset_index(drop = True)
    recession_start = recession_start['Quarter'].iloc[0]
    gdp['Recession Start Dummy'] = np.nan
    gdp['Recession Start Dummy'][gdp['Quarter'] == recession_start] = 1
    gdp['Recession Start Dummy'] = gdp['Recession Start Dummy'].ffill()
    gdp['Recession End Dummy'] = np.nan
    gdp['Recession End Dummy'][(gdp['Change in GDP'] > 0) & (gdp['Lagged Change in GDP'] > 0)] = 1
    recession_end = gdp[(gdp['Recession Start Dummy'] == 1) & (gdp['Recession End Dummy'] == 1)]
    recession_end = recession_end.reset_index(drop = True)
    recession_end = recession_end['Quarter'].iloc[0]
    return recession_end
#a = get_recession_end()
#print(a)
```

Return the year and quarter of the recession bottom time as a string value in a format such as 2005Q3

```
def get_recession_bottom():
    # Load data
    gdp = pd.read_excel('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/gdplev.xlsx', skiprows = 5)
    gdp = gdp[['Unnamed: 4', 'GDP in billions of chained 2009 dollars.1']]
    gdp.rename(columns = {'Unnamed: 4': 'Quarter', 'GDP in billions of chained 2009 dollars.1': 'GDP'}, inplace = True)
    i = gdp.index[gdp.Quarter == '2000Q1'][0]
    gdp = gdp[(gdp.index >= i)]
    gdp = gdp.reset_index(drop = True)
    # Get recession start and end and delete data outside of recession period
    start = get_recession_start()
    end = get_recession_end()
    gdp['Start'] = np.nan
    gdp.Start[gdp.Quarter == start] = 1
    gdp.Start = gdp.Start.ffill()
    gdp['End'] = np.nan
    gdp.End[gdp.Quarter == end] = 1
    gdp.End = gdp.End.bfill()
    recession = gdp[(gdp.Start == 1) & (gdp.End == 1)]
    recession = recession[recession.GDP == recession.GDP.min()]
    recession = recession.reset_index(drop = True)
    recession = recession.Quarter.iloc[0]
    return recession
#a = get_recession_bottom()
#print(a)
```

Convert the housing data to quarters and return it as mean values in a dataframe. 
This dataframe has columns for 2000Q1 through 2016Q3, and a multi-index in the shape of ["State", "RegionName"].

```
def convert_housing_data_to_quarters():
    housing_data = pd.read_csv('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/City_Zhvi_AllHomes.csv', header = 0)
    housing_data = housing_data.drop(['Metro', 'CountyName', 'SizeRank'], axis = 1)
    #print(housing_data.shape)
    x = housing_data.drop_duplicates(subset = 'RegionID', keep = 'first')
    #print(x.shape)    
    housing_data = housing_data.set_index(['State', 'RegionName', 'RegionID'], drop = True) 
    housing_data.columns.name = 'Month'
    x = housing_data.stack()
    x.name = 'Value'
    x = x.reset_index()
    x.Month = pd.to_datetime(x.Month)
    x['Year'] = x.Month.dt.year
    x['Quarter'] = x.Month.dt.quarter
    x = x.groupby(['State', 'RegionName', 'RegionID', 'Year', 'Quarter'],).mean()['Value']
    x = x.reset_index()
    #print(x.dtypes)
    x['Date'] = x.Year.apply(str) + 'Q' + x.Quarter.apply(str)
    x = x[(x.Year >= 2000)]
    x = x.drop(['Year', 'Quarter'], axis = 1)
    x = x.set_index(['State', 'RegionName', 'RegionID', 'Date'])
    x = x['Value']
    x = x.unstack('Date')
    x = x.reset_index()
    x.rename(columns = {'State': 'Code'}, inplace = True)

    # Merge university towns and state codes
    st_codes = pd.DataFrame.from_dict(states, orient = 'index')
    st_codes = st_codes.reset_index()
    st_codes.columns = ['Code', 'State']
    x = x.merge(st_codes, how = 'inner', on = 'Code')
    x = x.drop('Code', axis = 1)
    x = x.drop('RegionID', axis = 1)
    x = x.set_index(['State', 'RegionName'])
    
    return x
#a = convert_housing_data_to_quarters()
#print(a)
```

The analysis is shown [here](https://www.wikipedia.org/)

First create new data showing the decline or growth of housing prices between the recession start and 
the recession bottom. Then run a ttest comparing the university town values to the non-university towns values, 
return whether the alternative hypothesis (that the two groups are different) is true or not as well as 
the p-value of the confidence. 

Return the tuple (different, p, better) where different=True if the t-test is True at a p<0.01 
(we reject the null hypothesis), or different=False if otherwise (we cannot reject the null hypothesis). 
The value for better should be either "university town" or "non-university town" depending on which has 
a lower mean price ratio (which is equivilent to a reduced market loss).

```
def run_ttest():    
    # Create university towns dummy
    u_towns = get_list_of_university_towns()
    u_towns = u_towns[['State', 'RegionName']]
    u_towns['U_Town'] = 1
    u_towns = u_towns.set_index(['State', 'RegionName'])
    
    # Merge university towns with housing data by state code and region name
    x = convert_housing_data_to_quarters()
    x = x.merge(u_towns, how = 'left', left_index = True, right_index = True)
    x.U_Town[x.U_Town.isnull()] = 0
    
    # Analyze trends in housing values
    start = get_recession_start()
    bottom = get_recession_bottom()
    y = x[[start, bottom, 'U_Town']]
    y['Change'] = y['2009Q2'] / y['2008Q3']
    ut = y[y['U_Town'] == 1]['Change'].dropna()
    not_ut = y[y['U_Town'] == 0]['Change'].dropna()
    t = ttest_ind(ut, not_ut)
    p = t[1]
    different = False
    if p < 0.01:
        different = True
    better = 'university town'
    if ut.mean() < not_ut.mean():
        better = 'non-university town'
    print(ut.mean())
    print(not_ut.mean())
    answer = (different, p, better)
    return answer
a = run_ttest()
print(a)
```


Here is how you make [a link](https://eagronin.github.io/hello-world/).

