# Analyzing the Changing Distributions of Cyclist Casualties in New York City
## Introduction
I performed this analysis as part of the [Transportation Data Science Project (TDSP)](https://nebigdatahub.org/nsdc/tdsp/), which provides self-paced instruction on the basics of Python and data science using a New York City OpenData transportation dataset on car crashes. The project culminates with participants selecting an original research question, conducting data analysis to answer the question, and creating a poster to present their findings.

I decided to examine cyclist casualties (i.e., injuries and fatalities) before and after the COVID-19 pandemic to identify trends the pandemic may have caused and recommend actions to address them. This repo contains a [Python notebook](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/NYC-Cyclist-Casualties.ipynb) with the code for my analysis and my [research poster](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/Poster_NYC-Cyclist-Casualties_jgarties.pdf). This README walks through the code.
## Load Data
This analysis uses the NYC OpenData Motor Vehicles Collisions - Crashes dataset. Click the Export button on the page to download a CSV. 

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

![dataset head](https://raw.githubusercontent.com/jgarties/NYC-Cyclist-Casualties/refs/heads/main/screenshots/dataset_head.png "Dataset Head")
## Check for Missing Values
A large number of missing values could affect our ability to analyze this data. This code returns a table showing the number of missing values for each column and the percent of total values the missing values represent.
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
annual_cyc = cyc.groupby(cyc['CRASH DATE'].dt.to_period("Y")).agg({
    'COLLISION_ID': 'size',
    'NUMBER OF CYCLIST INJURED': 'sum',
    'NUMBER OF CYCLIST KILLED': 'sum'
}).rename(columns={'COLLISION_ID': 'Number of Crashes'})

# Filter the dataframe to only include the years 2017 - 2024
annual_cyc = annual_cyc[(annual_cyc.index >= '2017') & (annual_cyc.index <= '2024')]

# Calculate cyclist injury and fatality rates for each year and add them to the dataframe
annual_cyc['Cyclist Injury Rate'] = annual_cyc['NUMBER OF CYCLIST INJURED']/annual_cyc['Number of Crashes']
annual_cyc['Cyclist Fatality Rate'] = annual_cyc['NUMBER OF CYCLIST KILLED']/annual_cyc['Number of Crashes']

annual_cyc
```
Studying these results, we can observe the following:
- The _number_ of crashes per year is trending _down_, with a sharp drop in 2020.
- The _number_ of cyclists injured per year has remained relatively _flat_.
- The _number_ of cyclists killed per year has fluctuated. The small number of fatalities makes it difficult to identify a trend.
- The _rate_ of cyclists injured per year jumped in 2020 and generally _increased_ through 2024.
- The _rate_ of cyclists killed per year jumped in 2020 and fluctuated through 2024.

![summary of crashes, cyclist injuries, and cyclist fatalities, 2017-2024](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/summay_by_year.png?raw=true "Sumary by Year")
## Visualize Number of Crashes and Cyclist Casualties
We will visualize these results to help make more sense of them.
First, we will visualize the number of crashes per year, which shows the downward trend we observed.
```python
# Plot the number of crashes per year
plt.figure(figsize=(16, 6)) # Adjust figure size to meet your needs
sns.barplot(x=annual_cyc.index, y=annual_cyc['Number of Crashes'].values)
plt.title('Total Crashes per Year', fontsize=16)
plt.xlabel('Year', fontsize=14)
plt.ylabel('Number of Crashes', fontsize=14)
plt.tight_layout()

# plt.savefig("total_crashes_year.svg", format='svg') # Optional: Save the figure

plt.show()
```
![bar chart showing the total number of crashes per year](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/crashes_per_year.png?raw=true "Crashes per Year")

We will use two combo charts to visualize the trends in the number and rate of cyclists killed and injured. These charts will combine a bar chart of crashes injuring/killing cyclists using the left y-axis with a superimposed line chart of the rate of cyclists injured/killed using the right y-axis.

Cyclist injuries:
```python
# Create a figure
fig, ax1 = plt.subplots(figsize=(16,6)) # Adjust figure size to meet your needs

# Create a barplot for the number of cyclists injured
sns.barplot(x=annual_cyc.index.astype(str), y=annual_cyc['NUMBER OF CYCLIST INJURED'].values, ax=ax1)
ax1.set_ylabel('Number of Cyclists Injured', fontsize=14)
ax1.set_xlabel('Year', fontsize=14)

# Create the secondary y-axis
ax2 = ax1.twinx()

# Create a lineplot for cyclist injury rate
sns.lineplot(x=annual_cyc.index.astype(str), y=annual_cyc['Cyclist Injury Rate'].values, ax=ax2, color='red')
ax2.set_ylabel('Cyclist Injury Rate (all crashes)', fontsize=14)

# Set title and labels
plt.title('Cyclists Injured in Crashes', fontsize=16)

# plt.savefig("cyc_inj_rate_year.svg", format='svg') # Optional: Save the figure

plt.show()
```
Cyclist fatalities:
```python
# Create a figure
fig, ax1 = plt.subplots(figsize=(16, 6)) # Adjust figure size to meet your needs

# Create a barplot for the number of cyclists killed
sns.barplot(x=annual_cyc.index.astype(str), y=annual_cyc['NUMBER OF CYCLIST KILLED'].values, ax=ax1)
ax1.set_ylabel('Number of Cyclists Killed', fontsize=14)
ax1.set_xlabel('Year', fontsize=14)

# Create the secondary y-axis
ax2 = ax1.twinx()

# Create a lineplot for cyclist injury rate
sns.lineplot(x=annual_cyc.index.astype(str), y=annual_cyc['Cyclist Fatality Rate'].values, ax=ax2, color='red')
ax2.set_ylabel('Cyclist Fatality Rate (all crashes)', fontsize=14)

# Set title and labels
plt.title('Cyclists Killed in Crashes', fontsize=16)

# plt.savefig("cyc_fatality_rate_year.svg", format='svg') # Optional: Save the figure
plt.show()
```
This produces the following two charts:

![combo chart showing the number and rate of cyclists injured, 2014-2017](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/cyclists_injured_combo_chart.png?raw=true "Number and Rate of Cyclists Injured")

![combo chart showing the number and rate of cyclists killed, 2014-2017](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/cyclists_killed_combo_chart.png?raw=true "Number and Rate of Cyclists Killed")

## Heatmapping Cyclist Casualties Before and After the Pandemic
So far, we have seen an increase in the rate of cyclists killed and injured since 2020. This suggests that something about the change in travel patterns since the pandemic has caused a higher rate of cyclist casualties. Did these new travel patterns affect _where_ crashes injured and killed cyclists? We can use heatmapping to find out.

Returning to the full unsummarized dataset, we will create two heatmaps of crashes that killed and injured cyclists: one from 2017-2019, and one from 2022-2024. I selected these timeframes because they each provide three years of data to represent "pre-pandemic" and "post-pandemic" crashes. I omit the years 2020 and 2021 because they represent the greatest periods of pandemic disruption, while 2022 onward best represents the post-pandemic "new normal."

Heatmap for 2017-2019:
```python
# Drop rows with missing latitude and longitude values
data_geo = data.dropna(subset=['LATITUDE', 'LONGITUDE'])

# Drop rows where no cyclist was killed or injured
data_cyc = data_geo[(data_geo['NUMBER OF CYCLIST KILLED'] > 0) | (data_geo['NUMBER OF CYCLIST INJURED'] > 0)]

# Filter rows for desired years
data_cyc_17_19 = data_cyc[(data_cyc['CRASH DATE'] >= '2017-01-01') & (data_cyc['CRASH DATE'] <= '2019-12-31')]

# Create a base map
cyc_17_19 = folium.Map(location=[40.730610, -73.935242], zoom_start=10)

# Add heatmap
heat_cyc = [[row['LATITUDE'], row['LONGITUDE']] for index, row in data_cyc_17_19.iterrows()]
HeatMap(heat_cyc, radius=8, max_zoom=13).add_to(cyc_17_19)

# Save the map
cyc_17_19.save("cyc_heatmap_2017_2019.html")
```
Heatmap for 2022-2024:
```python
# Filter rows for desired years
data_cyc_22_24 = data_cyc[(data_cyc['CRASH DATE'] >= '2022-01-01') & (data_cyc['CRASH DATE'] <= '2024-12-31')]

# Create a base map
cyc_22_24 = folium.Map(location=[40.730610, -73.935242], zoom_start=10)

# Add heatmap
heat_cyc = [[row['LATITUDE'], row['LONGITUDE']] for index, row in data_cyc_22_24.iterrows()]
HeatMap(heat_cyc, radius=8, max_zoom=13).add_to(cyc_22_24)

# Save the map
cyc_22_24.save("cyc_heatmap_2022_2024.html")
```
![two heatmaps of cyclist casualties across New York City, one from 2017-2019 and one from 2022-2024](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/heatmaps.png?raw=true "Heatmaps of Cyclist Casualties")

The resulting heatmaps show that pre-pandemic (2017-2019, left) cyclist casualties were concentrated in Midtown and Lower Manhattan. Post-pandemic (2022-2024, right), they are more broadly distributed across the boroughs. Can we use this dataset to learn more about some of the post-pandemic hot spots?

## Mapping Post-Pandemic Hotspots
The dataset provides the latitude and longitude of crashes, allowing us to map precisely where crashes that injured or killed cyclists occurred to search for insights. However, while heatmapping can help us draw insights from a large amount of data, the sheer number of crashes in our dataset makes it challenging to learn anything from examining all crashes with cyclist casualties.

For example, this map of cyclist casualties from 2022-2024 uses yellow circles to mark the location of crashes that injured cyclists and red triangles to mark the location of crashes that killed cyclists. Even focusing on a small area like Williamsburg shows that there is too much data to gain useful insights.
```python
# Create a base map
cyc_casualty_22_24 = folium.Map(location=[40.730610, -73.935242], zoom_start=10)

# Add polygons for cyclist fatalities to map
for index, row in data_cyc_22_24.iterrows():
    if row['NUMBER OF CYCLIST KILLED'] > 0:
      color = "red"
      folium.features.RegularPolygonMarker(
          location=[row['LATITUDE'], row['LONGITUDE']],
          number_of_sides=3,
          radius=5,
          gradient = False,
          color=color,
          fill=True,
          fill_color=color
        ).add_to(cycfatality_22_24)
    elif row['NUMBER OF CYCLIST INJURED'] > 0:
      color = "yellow"
      folium.CircleMarker(
          location=[row['LATITUDE'], row['LONGITUDE']],
          radius=5,
          color=color,
          fill=True,
          fill_color=color
       ).add_to(cycfatality_22_24)

# Save the map
cyc_casualty_22_24.save("cyc_fatalities_2022_2024.html")
```
![a map of williamsburg showing locations of cyclist injuries and fatalities, using shape and color to differentiate between the types of accidents](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/casualties_williamsburg.png?raw=true "Williamsburg Cyclist Casualties, 2022-2024")

However, we can limit the map to only show cyclist fatalities. As the most serious crashes, these may provide illustrative examples of the factors contributing to higher rates of cyclist casualties.
```python
# Create a base map
cyc_fatality_22_24 = folium.Map(location=[40.730610, -73.935242], zoom_start=10)

# Add polygons for cyclist fatalities to map
for index, row in data_cyc_22_24.iterrows():
    if row['NUMBER OF CYCLIST KILLED'] > 0:
      color = "red"
      folium.features.RegularPolygonMarker(
          location=[row['LATITUDE'], row['LONGITUDE']],
          number_of_sides=3,
          radius=5,
          gradient = False,
          color=color,
          fill=True,
          fill_color=color
        ).add_to(cyc_fatality_22_24)

# Save the map
cyc_fatality_22_24.save("cyc_fatalities_2022_2024.html")
```
Examining two of the hotspots--Williamsburg/Long Island City (left) and Downtown Flushing (right)--we can see that many of the fatalities from 2022-2024 appeared near expressways.
![maps of Williamsburg and Long Island City and Downtown Flushing with cyclist fatalities marked](https://github.com/jgarties/NYC-Cyclist-Casualties/blob/main/screenshots/fatality_maps.png?raw=true "Williamsburg LIC and Downtown Flushing Fatalities")
## Conclusion
Further research targeting areas where cyclist casualties increased post-pandemic could determine the causes of these crashes and identify solutions. Causes may include traffic volume, vehicle speed, road design, and driver and cyclist behavior. 
Depending on the causes, the City could implement solutions near expressways such as re-engineering roads, increasing enforcement (including automated traffic enforcement), and building protected cycling infrastructure.
