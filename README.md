# Analyzing the Changing Distributions of Cyclist Casualties in New York City
## Introduction
I performed this analysis as part of the [Transportation Data Science Project (TDSP)](https://nebigdatahub.org/nsdc/tdsp/), which provides self-paced instruction on the basics of python and data science using a New York City OpenData transportation dataset on car crashes. The project culminates with participants selecting and original research question, conducting data analysis to answer the question, and creating a poster to present their findings.

I decided to examine cyclist causialties (i.e., injuries and fatalities) before and after the COVID-19 pandemic to identify trends that the pandemic may have caused and recommend actions to address them. This repo contains a python notebook with the code for my analysis and my research poster. This readme walks through the code.
## Load Data
This analysis uses the NYC OpenData Motor Vehicles Collisions - Crashes dataset. Click the Export button on the page to download a csv. 

Once you've downloaded the data, import the libraries we'll be using in the analysis.
```python
# Import libraries
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import folium
from folium.plugins import HeatMap
```
```python
# Optional: If saving csv in Google Drive, mount your drive
from google.colab import drive
drive.mount('/content/drive')
```
Next, read the csv.
```python
# Read data from csv
data = pd.read_csv('filepath') # Replace filepath with the path to your csv

# View the column headers and the first five rows of the dataset
data.head(5)
```
This is my dataset's head. Yours may look different depending on when you downloaded the csv, but the column headers should be the same.

![dataset head](https://raw.githubusercontent.com/jgarties/NYC-Cyclist-Casualties/refs/heads/main/screenshots/dataset_head.png "Dataset Head")
## Check for Missing Values
A large number of missing values could affect our ability to analyze this data. This code returns a table showing the number of missing values for each column and the percent of the total values the missing values represent.
```python
# Find the number of missing values in each column
missing_values = data.isnull().sum()

# Calculate percentages of missing values
missing_values_percentage = (missing_values / len(data)) * 100

# Return counts and percentages of missing values in each column
missing_data = pd.DataFrame({'Missing Values': missing_values, 'Percentage (%)': missing_values_percentage})
missing_data.sort_values(by='Percentage (%)', ascending=False)
```
The table is below, with columns we will use in this analysis highlighted. Because these columns either have no missing values or a small percentage of missing values, we can proceed.

![missing values in dataset](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/missing_values.png?raw=true "Missing Values")
## Summarize Data
We will summarize the data by year to get the total number of crashes, cyclist injuries, and cyclist fatalities for each year. We will also calculate the rate of crashes that resulted in cyclist injuries and fatalities each year. Finally, we will filter the data to only include the years that are in scope (2017-2024).
```python
# Convert 'CRASH DATE' to datetime format
data['CRASH DATE'] = pd.to_datetime(data['CRASH DATE'])

# Create a dataframe of collisions, cyclist injuries, and cyclist fatalities
cyc = data[['CRASH DATE','COLLISION_ID','NUMBER OF CYCLIST INJURED','NUMBER OF CYCLIST KILLED']]

# Group by year to get the number of crashes per year
annual_cyc = pd.pivot_table(cyc, values=['COLLISION_ID','NUMBER OF CYCLIST INJURED','NUMBER OF CYCLIST KILLED'],
                                aggfunc={'COLLISION_ID':'size','NUMBER OF CYCLIST INJURED':'sum','NUMBER OF CYCLIST KILLED':'sum'},
                                index=cyc['CRASH DATE'].dt.to_period("Y"))
# Rename the COLLISION_ID column, now that it is the sum of the number of rows, i.e., crashes
annual_cyc = annual_cyc.rename(columns={'COLLISION_ID': 'Number of Crashes'})

# Filter the dataframe to only include the years 2017 - 2024
annual_cyc = annual_cyc[(annual_cyc.index >= '2017') & (annual_cyc.index <= '2024')]

# Calculate cyclist injury and fatality rates for each year and add them to the dataframe
annual_cyc['Cyclist Injury Rate'] = annual_cyc['NUMBER OF CYCLIST INJURED']/annual_cyc['Number of Crashes']
annual_cyc['Cyclist Fatality Rate'] = annual_cyc['NUMBER OF CYCLIST KILLED']/annual_cyc['Number of Crashes']

annual_cyc
```
Studying these results, we can observe the following:
- The _number_ of crashes per year is trending _down_, with a sharp drop in 2020
- The _number_ of cyclists injured per year has remained relatively _flat_
- The _number_ of cyclists killed per year has fluctuated. The small number of fatalities makes it difficult to identify a trend
- The _rate_ of cyclists injured per year jumped in 2020 and generally _increased_ through 2024
- The _rate_ of cyclists killed per year jumped in 2020 and fluctuated through 2024

