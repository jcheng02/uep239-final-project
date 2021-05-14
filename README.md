# Suitability Analysis: Best ZCTA Within the Boston Area to Live Without a Car 
**Tufts University\
Department of Urban and Environmental Policy and Planning\
UEP-239: Geospatial Programming with Python**\
Final Project\
By: Justina Cheng\
May 2021

---

# Abstract
## Overview
The purpose of this suitability analysis is to find the best Zip Code Tabulation Area (ZCTA) within the Boston Area for a Tufts University student and a Boston University student to live together without a car (i.e. somewhere equally convenient for both). Though the self-serving nature of the study is specific to students of Tufts and BU, a similar workflow can be conducted as a general analysis of the Boston Area (by omitting the network analysis between Tufts and BU) or by specifying different locations of interest (e.g. HubSpot's Cambridge office and the Museum of Fine Arts). The workflow can also be applied to any city or region, provided data is available.

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

## Miscellaneous
Choropleth colormaps were standardized based on the type of data to make them easily distinguishable: averages were mapped with `OrRd`, densities were mapped with `YlGnBu`, and reclassified data were mapped with `plasma`. 

---

# Instructions to Run Notebook
To run this notebook, first create the `suitability` environment from the `environment.yml` in the directory.

# Data Used

```{list-table}
:header-rows: 1
:widths: 10 20 5 5 5 5

* - Filename
  - Description
  - Format
  - Feature
  - CRS
  - Source
* - `CHCS_PT`
  - Massachusetts Community Health Centers
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-community-health-centers)
* - `COLLEGES_PT`
  - Massachusetts Colleges and Universities
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-colleges-and-universities)
* - `FARMERSMARKETS_PT`
  - Massachusetts Farmers Markets
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-farmers-markets)
* - `FIRESTATIONS_PT_MEMA`
  - Massachusetts Fire Stations
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-fire-stations)
* - `HOSPITALS_PT`
  - Massachusetts Acute Care Hospitals
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-acute-care-hospitals)
* - `HYDRO25K_POLY`
  - Massachusetts Surface Water
  - ESRI Shapefile
  - Polygon
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-massdep-hydrography-125000)
* - `LIBRARIES_PT`
  - Massachusetts Public Libraries
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-libraries)
* - `MBTA_ARC`
  - MBTA Rapid Transit Lines
  - ESRI Shapefile
  - Line
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-rapid-transit)
* - `MBTABUSSTOPS_PT`
  - MBTA Bus Stops
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-bus-routes-and-stops)
* - `MBTABUSROUTES_ARC`
  - MBTA Bus Routes
  - ESRI Shapefile
  - Line
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-bus-routes-and-stops)
* - `MBTA_NODE`
  - MBTA Rapid Transit Stops
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-mbta-rapid-transit)
* - `NEWMASK_POLY`
  - New England Mask Around Massachusetts
  - ESRI Shapefile
  - Polygon
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-new-england-boundaries)
* - `OUTLINE25K_POLY`
  - Massachusetts Outline with Detailed Coastline
  - ESRI Shapefile
  - Polygon
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-state-outlines)
* - `TRAINS_NODE`
  - Massachusetts Train Stations
  - ESRI Shapefile
  - Point
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-trains)
* - `TRAINS_RTE_TRAIN`
  - MBTA Commuter Rail Routes
  - ESRI Shapefile
  - Line
  - [EPSG:26986](https://epsg.io/26986)
  - [MassGIS](https://docs.digital.mass.gov/dataset/massgis-data-trains)
```  
  
# Packages Used
  - numpy
  - pandas
  - shapely
  - geopandas
  - geopy
  - geojson
  - bokeh
  - matplotlib
  - seaborn
  - folium
  - mplleaflet
  - networkx
  - osmnx

# Acknowledgements

# References

