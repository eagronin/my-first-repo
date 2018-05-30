# Data Acquisition

## Overview
This section describes the data acquisition process for testing a hypothesis whetherd university towns have their mean housing prices less effected by recessions.  This analysis requires a list of university towns, mean / median housing prices by city for the purpose of comparing housing prices in university towns versus non-university towns, and GDP growth for determining the timing of recessions.

The next step, which includes cleaning and processing of the data, is described [here](https://eagronin.github.io/university-towns-prepare/).

## Data 
All the data have been downloaded from the Coursera website. The original sources of the data and the code for loading are described below.

A list of university towns in the United States is available from the Wikipedia page on college towns.  A university town is a city which has a high percentage of university students compared to the total population of the city.  The following function loads a list of university towns and states in which these towns are located from a text file into a pandas data frame.

```
def load_university_town_data():
    university_towns = pd.read_table('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/university_towns.txt', header = None)
    university_towns.columns = ['RegionName']
    return university_towns
```

The GDP over time of the United States in current dollars (the chained value in 2009 dollars), in quarterly intervals, is from Bureau of Economic Analysis, US Department of Commerce,  in the file gdplev.xlsx. 
A quarter is a specific three month period, Q1 is January through March, Q2 is April through June, 
Q3 is July through September, Q4 is October through December.
GDP data used in the analysis are from the first quarter of 2000 onward.  The following function loads the GDP data from an Excel file into a pandas data frame and drops the data that are not used in the analysis:  

```
def load_gdp_data():
    gdp = pd.read_excel('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/gdplev.xlsx', skiprows = 5)
    gdp = gdp[['Unnamed: 4', 'GDP in billions of chained 2009 dollars.1']]
    gdp.rename(columns = {'Unnamed: 4': 'Quarter', 'GDP in billions of chained 2009 dollars.1': 'GDP'}, inplace = True)
    i = gdp.index[gdp.Quarter == '2000Q1'][0]
    gdp = gdp[(gdp.index >= i)]
    gdp = gdp.reset_index(drop = True)
    return gdp
```

Housing data for the United States is from the Zillow research data site.  In particular the datafile for all homes at a city level, City_Zhvi_AllHomes.csv, has median home sale prices at a fine grained level.  The following function loads the housing data from a CSV file into a pandas data frame:

```
def load_housing_data():
    housing_data = pd.read_csv('/Users/eagronin/Documents/Data Science/Portfolio/Project Data/City_Zhvi_AllHomes.csv', header = 0)
    housing_data = housing_data.drop(['Metro', 'CountyName', 'SizeRank'], axis = 1)
    return housing_data
```

The following dictionary was used to map state names to two letter acronyms:

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

The next step, which includes cleaning and processing of the data, is described [here](https://eagronin.github.io/university-towns-prepare/).
