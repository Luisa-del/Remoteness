# Remoteness analysis

This is a step-by-step tutorial on how to perform a remoteness analysis for user-defined areas. The tutorial is split into two parts:
1. [Prepare input data using QGIS (optionally R)](#1-Prepare-input-data) <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_r.png"> <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_qgis.png">
2. [Perform remoteness analysis in Google Earth Engine (GEE)](#2-perform-remoteness-analysis) <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_gee.png">

## Overview

* Remoteness indicates the distance to roads, but takes into account the cost to walk through terrain. It can serve as anthropogenic factor in wildlife studies, where it has been shown to be an important driver of species occurrence.
  
* Costs are calculated from a hiking function that considers slope. The function is also defined to take metabolic costs into account, because it has been shown that humans are typically not following least-time routes, especially in mountainous areas.
  
* Drivable roads can serve as starting points from where the costs are then added up (cumulative costs) in order to assess remoteness. This can be points along drivable roads from OpenStreetMap (osm), or GPS points.

* The drivable roads should cover a certain buffer area around the study area. This ensures that roads in the vicinity of the study area are also included in the calculation of remoteness and avoids distorted remoteness values at the border of the study area.

* Some areas like water bodies or other inaccessible areas affect accessibility and can be masked out prior to the analysis.

![](".png")

*Figure: Remoteness layer for province of Kâmpóng Thum (Cambodia), using osm drivable roads and GLobal Surface Water mask.*
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/remoteness.png">

![](".png")

**Useful information before performing remoteness analysis for own study area**

1. **Store AOI as shape file.** You should save your study area as a shape file on your local computer. [GADM](https://gadm.org/download_country.html) provides free and easy accessible country data at different subdivision levels ------------------------------(see section ...).
  
2. **Apply sufficient buffer around study area.** A buffer area of 10 km around the study area was considered large enough to ensure that roads that have a relevant influence on the remoteness values at the border of the AOI are included in the analysis. The buffer value can be modified by the user ---------------------------(see section ...).

3. **Choose software to download & prepare osm data (size dependent!).** You need to try what's best for you. The larger the study area and amount of data, the longer the processing!
    + Simplest is to use the **"QuickOSM-plugin"** from QGIS for *smaller* areas like National Parks or province borders. Here you can download the data already at the size of your buffered AOI and filter it by drivable road categories -------------------(see section ...). If the study area is *too large* for QuickOSM (like a whole country), either use smaller tiles in QuickOSM,
    + or download and prepare the data in R. Here you can download the data on a country level, filter by drivable road categories and then optionally further clip or merge to your buffered AOI -----------------------------(more info see section ...). 

4. **If possible (!) verify osm roads.** OpenStreetMap is a community-based, freely available, editable map service and provides free and open source geographical data sets of natural or man made features. Given that it is edited mainly by volunteers with different mapping skills, the completeness and quality of its annotations are heterogeneous across different geographical locations, and updates are more regularly in urban areas than in rural areas. If possible, verify whether the selected road data represents the road network of the study area as good as possible. 

5. **Convert osm drivable roads to starting points.** Use the **"Points along geometry"-tool** from QGIS to convert drivable road lines to equally spaced points. Sufficient spacing should be 50 or 100 meters.

6. **Carefully select data for GEE upload.** Only upload starting points and study area (not buffered study area!). Apply the same buffer in the GEE script that was used to create the starting points (see section 4).

7. **GPS points available?** In case you want to use other starting point for your analysis (e.g GPS points along the road network, villages,...) you only need to upload your study area and the starting point dataset to GEE. Consider a buffer!

![](".png")

**In the tutorial below, the province of Kâmpóng Thum (Cambodia) will be our study area. For your remoteness analysis, select only the steps that are relevant based on the data you have / need.**


![](".png")

## 1. Prepare input data

### 1.1 Get shapefile of study area

GADM provides spatial data for all countries and their sub-divisions. You can download your required country shape file either from their [website](https://gadm.org/download_country.html) or inside R.

```{r}
### Define parameters

# Set path to working directory
wd <- "D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial"

# For GADM download set name, iso code and level for required country
name <- "cambodia"
iso <- "KHM"
level <- 1

# Set buffer that should be applied to aoi
buffer <- 10000

# Set EPSG code of a metric CRS (e.g utm)
crs_m <- 32648
```
  
```{r}
### Download country data 

# Get or import country data
country <- geodata::gadm(iso, level=level, path = tempdir())
country <- st_as_sf(country)

# # Export to folder
# st_write(country,
#          dsn = file.path(wd, "data/gadm/"),
#          layer = paste0("gadm_", name, "_", level),
#          driver = "ESRI Shapefile")
```
  
```{r}
### Select or import aoi

# Select province as example aoi, or import your Area of Interest
province <- "Kâmpóng Thum"
aoi <- country[(country$NAME_1 %in% province),]

# Export to folder
# st_write(aoi,
#          dsn = file.path(wd, "data/gadm/"),
#          layer = "aoi",
#          driver = "ESRI Shapefile")

```

```{r class.source = 'fold-hide'}
# Use mapview for interactive mapping
mapview(country, layer.name = name) + mapview(aoi, col.regions = "orange")
```
*Figure: Province Kâmpóng Thum (study area) in cambodia.*
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/aoi.PNG">

![](".png")

### 1.2 Get OpenStreetMap data

**Option 1: download data via QuickOSM plugin in QGIS**  
  
* Only possible for *smaller* study areas like national parks or provinces. Not possible for whole countries - QGIS crashes if AOI is too large!
* When choosing that option, you can already filter the required drivable road categories manually in QGIS and skip section 3.1 - 3.4 in this tutorial.

1. Import your study area to QGIS, open the buffer tool and create a 10 km buffer around it. Therefore, the layer should be stored with a metric coordinate system, like in this case UTM zone 48N (EPSG: 32648) or Pseudo-Mercator (EPSG: 3857).
2. Enable or install QuickOSM plugin and open it (click on magnifying glass icon).
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_quickosm1.png">

3. 
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_quickosm2.png">














## 2. Perform remoteness analysis




