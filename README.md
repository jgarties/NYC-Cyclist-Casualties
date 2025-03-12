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
