# Suitability Analysis: Best ZCTA Within the Boston Area to Live Without a Car 
**Tufts University<br>
Department of Urban and Environmental Policy and Planning<br>
UEP-239: Geospatial Programming with Python**<br>
Final Project<br>
By: Justina Cheng<br>
May 2021

---

# Abstract
## Overview
The purpose of this suitability analysis is to find the best Zip Code Tabulation Area (ZCTA) within the Boston Area for a Tufts University student and a Boston University student to live together without a car (i.e. somewhere equally convenient for both). Though the self-serving nature of the study is specific to students of Tufts and BU, a similar workflow can be conducted as a general analysis of the Boston Area (by omitting the network analysis between Tufts and BU) or by specifying different locations of interest (e.g. HubSpot's Cambridge office and the Museum of Fine Arts). The workflow can also be applied to any city or region, provided the necessary data is available.

The study is structured as follows:
1. Import dependencies
    - Necessary packages can be found in the `environment.yml` file.
1. Create a **base map** of the region.
1. Import and view mass transit stops and routes.
1. Limit the study area to the extent of rapid transit lines.
1. Conduct a **shortest path network analysis** between Tufts and BU to use as visual reference points for areas that are convenient across indicators for both a Tufts student and a BU student. 
1. Analyze and reclassify six indicators necessary for living without a car:
    - **Mass transit accessibility** using density of stops.
    - **Rent affordability** using median rental prices for a two- or three-bedroom home.
    - Necessities:
        - **Food availability** using density of food establishments (e.g. supermarkets, restaurants, etc.).
        - **Health services availability** using density of healthcare establishments (e.g. doctors' offices, clinics, etc.).
        - **Public services availability** using density of public services (e.g. post offices, fire stations, etc.).
     - **Leisure potential** using density of leisure establishments (e.g. parks, sports facilities, museums, etc.).
1. Calculate a ***weighted suitability index*** based on individual importance of the six indicators.
1. Analyze results of the suitability index, compare to the locations of and shortest paths between Tufts and BU, and determine which ZCTAs are best suited.

The analysis concluded that ZCTAs 02145 and 02143 were the best places for a Tufts University student and a Boston University student to live together without a car. They are tied for second in overall suitability (3.925) and are on or in between the shortest paths between Tufts and BU. The ultimate choice for housing thereby relies on whether the students value lower rents (02145 rent score is 4, 02143 rent score is 3) or density of mass transit, health services, and public services (02143 exceeds 02145 on all three indicators).

## Functions Created
- `convert_n_clip` takes two GeoDataFrames (GDF): one to process (gdf) and one whose extent will be used to clip. The function converts the coordinate reference system (CRS) of the original GDF and clips it to the extent of the extent GDF.
- `read_n_clip` takes a filepath for a shapefile and a GDF whose extent will be used to clip the shapefile. The function reads in the shapefile and uses `convert_n_clip` to convert the coordinate reference system (CRS) of the original GDF and clip it to the extent of the extent GDF.
- `count_records` takes a GeoDataFrame and counts the number of records within another GDF of polygons. It outputs a DataFrame with the specified polygon column values and counts column. The optional argument `op` for `gpd.sjoin` defaults to `'within'` unless otherwise specified.
- `multimerge` takes a base DataFrame and merges it with each DataFrame in a list of DataFrames on the specified column or list of columns and with the specified `how`. The function assumes the specified column(s) exist(s) across all DataFrames.
- `quantiles` takes a DataFrame, a column name, and a list of quantile thresholds (between 0 and 1) and outputs a list of quantile values for the column.
- `reclass_5` takes a value and reclassifies it into 1 of 5 classes given a list of values for class thresholds and order preference.
- `calc_density` automates density calculations based on counts of records in multiple datasets (e.g. density of food establishments per square kilometer in each ZCTA based on groceries, prepared food, and farmers markets). It takes a base GeoDataFrame and, using function `multimerge`, merges it with a list of GeoDataFrames on the specified column or columns in the list of `on` columns and with the specified `how`. The GDFs within the list are expected to have columns with counts, and those columns are given in a list. The function then adds together the counts in specified columns and calculates the density.

## Notes
### Boston Region vs. Boston Area
*Boston Region* was used to reference the Boston Region MPO. *Boston Area* was used to distinguish the limited extent.

### Rental Data
Data was obtained from [Jeff Kaufman's Apartment Price Map](https://www.jefftk.com/apartment_prices/details), which scrapes data from [Padmapper](https://www.padmapper.com/). Though the dataset does not cover the entire defined extent (53 of 62 ZCTAs are represented), it was the best source of point data found for the Boston Region.

Zillow data was also considered as a source, but it was much more limited in extent. Only 37 of the 62 ZCTAs in question were available in the Zillow dataset. Zillow analysis has been included as an appendix at the end of the study for those who are curious.

### Colormaps
Choropleth colormaps were standardized based on the type of data to make them easily distinguishable: averages were mapped with `OrRd`, densities were mapped with `YlGnBu`, and reclassified data were mapped with `plasma`. 

---
---

# Instructions to Run Notebook
## Run Locally
Anaconda or Miniconda are required to run this notebook locally.
1. Download all files within the repository. 
1. In Terminal, navigate to the folder location.
1. Create the `suitability` environment from the `environment.yml` file.
    > `mamba env create -f environment.yml`
1. Activate the environment 
    > `conda activate suitability`
1. Open Jupyter Lab
    > `jupyter lab`
1. Open the `.ipynb` file and go through cell by cell.

## Run in Binder
1. Go to this [Binder link](https://mybinder.org/v2/gh/jcheng02/uep239-final-project/HEAD) and wait for it to load.
1. Open `jcheng02_uep239_final_project.ipynb`.
1. Restart kernel and run.

---
---

# Data Used
All data used was processed within the Jupyter Notebook.

## Base Map
Filename | Description | Format | Feature | CRS | Source 
-------- | ----------- | ------ | ------- | --- | ------
`MPO_Boundaries` | Boundaries of Massachusetts Metropolitan Planning Organizations | ESRI Shapefile | Polygon | [EPSG:3857](https://epsg.io/3857) | [MassDOT](https://geo-massdot.opendata.arcgis.com/datasets/mpo-boundaries)
`tl_2010_25_zcta510` | 2010 Massachusetts 5-Digit Zip Code Tabulation Area (ZCTA) Boundaries | ESRI Shapefile | Polygon | [EPSG:4269](https://epsg.io/4269) | [Census Bureau](https://www.census.gov/cgi-bin/geo/shapefiles/)
`HYDRO25K_POLY` | Massachusetts Surface Water | ESRI Shapefile | Polygon | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-massdep-hydrography-125000)
`NEWMASK_POLY` | New England Mask Around Massachusetts | ESRI Shapefile | Polygon | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-new-england-boundaries)
`OUTLINE25K_POLY` | Massachusetts Outline with Detailed Coastline | ESRI Shapefile | Polygon | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-state-outlines)

## Mass Transit
Filename | Description | Format | Feature | CRS | Source 
-------- | ----------- | ------ | ------- | --- | ------
`MBTABUSSTOPS_PT` | MBTA Bus Stops | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-bus-routes-and-stops)
`MBTABUSROUTES_ARC` | MBTA Bus Routes | ESRI Shapefile | Line | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-bus-routes-and-stops)
`MBTA_ARC` | MBTA Rapid Transit Lines | ESRI Shapefile | Line | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-rapid-transit)
`MBTA_NODE` | MBTA Rapid Transit Stops | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-rapid-transit)
`TRAINS_NODE` | Massachusetts Train Stations | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-trains)
`TRAINS_RTE_TRAIN` | MBTA Commuter Rail Routes | ESRI Shapefile | Line | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-trains)

## Colleges
Filename | Description | Format | Feature | CRS | Source 
-------- | ----------- | ------ | ------- | --- | ------
`COLLEGES_PT` | Massachusetts Colleges and Universities | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-colleges-and-universities)

## Rental Data
Filename | Description | Format | Feature | CRS | Source 
-------- | ----------- | ------ | ------- | --- | ------
`20200919_rental_data.csv` | Padmapper Rental Data from 9/19/2020 | CSV file | Point | [EPSG:4326](https://epsg.io/4326) | [Jeff Kaufman's Apartment Price Map](https://www.jefftk.com/apartment_prices/details)
`Zip_ZORI_AllHomesPlusMultifamily_SSA.csv` | Zillow Rental Data | CSV file | N/A (zipcodes) | N/A | [Zillow Housing Data](https://www.zillow.com/research/data/)

## Necessities
Necessities not obtained via `OSMnx`
Filename | Description | Format | Feature | CRS | Source 
-------- | ----------- | ------ | ------- | --- | ------
`CHCS_PT` | Massachusetts Community Health Centers | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-community-health-centers)
`FARMERSMARKETS_PT` | Massachusetts Farmers Markets | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-farmers-markets)
`FIRESTATIONS_PT_MEMA` | Massachusetts Fire Stations | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-fire-stations)
`HOSPITALS_PT` | Massachusetts Acute Care Hospitals | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-acute-care-hospitals)
`LIBRARIES_PT` | Massachusetts Public Libraries | ESRI Shapefile | Point | [EPSG:26986](https://epsg.io/26986) | [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-libraries)
`USPS DDUs.csv` | USPS Destination Delivery Units (DDUs) in a specified map extent | CSV file | Point | [EPSG:3857](https://epsg.io/3857)] | [USPS](https://uspstools.maps.arcgis.com/apps/webappviewer/index.html?id=1fc1c26bb31246b39087606c65b83020)

---
---
  
# Packages Used
 - `numpy`: computing and working with arrays
 - `pandas`: data analysis and management
 - `shapely`: manipulation and analysis of planar geometric objects
 - `geopandas`: geospatial data analysis and management
 - `matplotlib`: plotting and visualization
 - `seaborn`: data visualization
 - `folium`: interactive maps
 - `networkx`: network analysis
 - `osmnx`: work with networks in OpenStreetMap

---
---

# Acknowledgements
This suitability analysis was created as a final project for Tufts University's UEP-239: Geospatial Programming with Python. I would like to thank the following members of the Tufts UEP community:
- Uku-Kaspar Uustalu for his tireless efforts in making the class a productive and pleasant experience for the students. His dedication to ensuring students understood Python in both theory and practice was crucial to the success of each student. 
- Ana Arsovska, Teaching Assistant, for providing guidance both during and outside of office hours and giving valuable feedback on assignments.
- Sumeeta Srinivasan for facilitating a restructuring of the course and syllabus to ensure content would remain comprehensive after unforeseen circumstances forced an adjustment.
- Kyle Monahan and Chris Barnett for stepping in as Guest Instructors and introducing students to different perspectives on Python.

---
---

# References
- [PEP 8 -- Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/#comments)
- [NumPy documentation](https://numpy.org/doc/stable/)
- [Pandas documentation](https://pandas.pydata.org/docs/user_guide/index.html#user-guide)
- [Shapely documentation](https://shapely.readthedocs.io/en/stable/manual.html)
- [GeoPandas documentation](https://geopandas.org/docs/user_guide.html)
- [Matplotlib documentation](https://matplotlib.org/stable/contents.html)
- [Seaborn documentation](https://seaborn.pydata.org/api.html)
- [Folium documentation](https://python-visualization.github.io/folium/)
- [NetworkX documentation](https://networkx.org/documentation/stable/)
- [OSMnx documentation](https://osmnx.readthedocs.io/en/stable/osmnx.html)
- [EPSG.io](http://epsg.io/)
- [OpenStreetMap: Map features](https://wiki.openstreetmap.org/wiki/Map_features)
- [Font Awesome: the Icons](https://fontawesome.com/v4.7.0/icons/)
- [GitHub: Mastering Markdown Guide](https://guides.github.com/features/mastering-markdown/)
- [Anita Graser: Interactive plots for GeoPandas GeoDataFrames of LineStrings](https://anitagraser.com/2019/10/31/interactive-plots-for-geopandas-geodataframe-of-linestrings/)
- [towards data science: Choropleth maps with Folium](https://towardsdatascience.com/choropleth-maps-with-folium-1a5b8bcdd392)
- [GitHub Pages](https://pages.github.com/)
- UEP-239 Homework Assignments 
- UEP-239 Class Exercises
- UEP-239 Lectures
