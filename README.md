# Remoteness analysis

This is a step-by-step tutorial on how to perform a remoteness analysis for user-defined areas. The tutorial is split into two parts:
1. [Prepare input data](#1Prepare-input-data-in-QGIS)
    1. [using QGIS](#1Prepare-input-data-in-QGIS)  <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_qgis.png" width="20">
    2. [using R](#1Prepare-input-data-in-R) <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_r.png"> 
2. [Perform remoteness analysis in Google Earth Engine (GEE)](#2perform-remoteness-analysis) <img src="https://github.com/Luisa-del/Remoteness/blob/main/img/logo_gee.png" width="20">

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

1. **Store AOI as shape file.** You should save your study area as a shape file on your local computer. [GADM](https://gadm.org/download_country.html) provides free and easy accessible country data at different subdivision levels.
  
2. **Apply sufficient buffer around study area.** A buffer area of 10 km around the study area was considered large enough to ensure that roads that have a relevant influence on the remoteness values at the border of the AOI are included in the analysis. The buffer value can be modified by the user.

3. **Choose software to download & prepare osm data (size dependent!).** You need to try what is more suitable for you and your analysis. The larger the study area and amount of data, the longer the processing!
    + Simplest is to use the **"QuickOSM-plugin"** from QGIS for *smaller* areas like national parks or province borders. Here you can download the data already at the size of your buffered AOI and filter it by drivable road categories (see [section 1](#1-Prepare-input-data-in-QGIS)). If the study area is *too large* for QuickOSM (like a whole country), you can split it into smaller tiles, run it again for each tile, and merge the osm data later on.
    + Alternatively download and prepare the data in R. Here you can download the data on a country level, filter it by drivable road categories and then optionally further clip or merge to your buffered AOI ([see section 2](#1-Prepare-input-data-in-R)). 

4. **If possible (!) verify osm roads.** OpenStreetMap is a community-based, freely available, editable map service and provides free and open source geographical data sets of natural or man made features. Given that it is edited mainly by volunteers with different mapping skills, the completeness and quality of its annotations are heterogeneous across different geographical locations, and updates are more regularly in urban areas than in rural areas. If possible, verify whether the selected road data represents the road network of the study area as good as possible. 

5. **Convert osm drivable roads to starting points.** Use the **"Points along geometry"-tool** from QGIS to convert drivable road lines to equally spaced points. Sufficient spacing should be 50 or 100 meters.

6. **Carefully select data for GEE upload.** Only upload starting points and study area (not buffered study area!). Apply the same buffer in the GEE script that was used to create the starting points (see section 4).

7. **GPS points available?** In case you want to use other starting point for your analysis (e.g GPS points along the road network, villages,...) you only need to upload your study area and the starting point dataset to GEE. Consider a buffer!

![](".png")

**In the tutorial below, the province of Kâmpóng Thum (Cambodia) will be our study area. For your remoteness analysis, select only the steps that are relevant based on the data you have / need.**


![](".png")

## 1. Prepare input data in QGIS

### 1.1 Download study area

GADM provides spatial data for all countries and their sub-divisions. You can download your required country shape file either from their [website](https://gadm.org/download_country.html) 

![](".png")

### 1.2 Download osm data

Download OpenStreetMap (osm) data via **"QuickOSM"-plugin** in QGIS. However, this is only possible for *smaller* study areas like national parks or provinces. Not possible for whole countries - QGIS crashes if AOI is too large!

1. Import your study area to QGIS, open the buffer tool and create a 10 km buffer around it. Therefore, the layer should be stored with a metric coordinate system, like in this case UTM zone 48N (EPSG: 32648) or Pseudo-Mercator (EPSG: 3857).
2. Enable or install QuickOSM plugin and open it (click on magnifying glass icon).
3. Then choose your buffered (!) AOI (1), select the osm key Highway/Streets (2), and from the listed elements remove all categories that are not drivable by car or motorcycle (3).
4. Open the *Advanced*-tab and only select way, lines, and multiline-strings. This can avoid errors.
5. Then click "Run query" (4) and the filtered osm road will be downloaded and imported to the QGIS project as a temporal layer. Optionally check the attribute table, and save the file on your local computer in a metric coordinate system (here UTM, EPSG 32648). This is important for the next step!

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_quickosm1.png">
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_quickosm2.png">

![](".png")

### 1.3 Convert lines to points 

Osm roads are multiline features, but the algorithm that calculates remoteness requires starting points. Therefore, lines need to be converted to equal spaced points. Using the **"Points along geometry"-tool** in QGIS is the easiest and fastest way to perform this step. The distance between the points was set to 100 meter in this tutorial.

1. Open the tool and select the filtered osm road dataset with a metric coordinate system.
2. Set the distance parameter and save file to your local computer.

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_100mpoints_1.png">
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_100mpoints_2.png">
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/qgis_100mpoints_3.png">



## 2. Prepare input data in R

### 2.1 Download study area

GADM provides spatial data for all countries and their sub-divisions. You can download your required country shape file from R.

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

### 2.2 Download osm data

tba...


### 2.3 Convert lines to points 

Import the filtered osm roads to QGIS and follow instructions from [here](#13-Convert-lines-to-points)

![](".png")



## 4. Perform remoteness analysis

* Google Earth Engine (GEE) is a cloud computing platform, powered by Google cloud infrastructure. It offers new opportunities for Big Data analyses with huge data sets up to petabyte scale, and analyses are run on Google servers, which have much higher computational power and storage capacity than most local computer sources.  
  
* The remoteness analysis can only be performed in GEE. Users must log in with a Google account in order to work with the user interface (uploading input file, running the script, exporting products). Hence it is not open source software but costfree.
  
* More information on how to create a Google Earth Engine account can be found [here](https://developers.google.com/earth-engine/guides/getstarted).  
* To get familiar with the user interface check out [this page](https://developers.google.com/earth-engine/guides/playground).  
  
**Useful information on script and analysis performance in GEE:**
  
* The script is designed in a user friendly way, because only the study area and the access points have to be imported by the user in the beginning.  
* The script allows the exports of the cost raster with the unit pace (hour per meter) and the cumulative cost (remoteness) raster with the unit in hours.  
* The larger the study area, the longer it can take to export the data.  
  

>[Follow this link to open the remoteness analysis script in Google Earth Engine.](https://code.earthengine.google.com/a0675d2189dc63bee738b16d84e18ec9)


### 4.1 Run demo

Open script link and "Run" demo to see how script performs remoteness analysis with example aoi and access points.

GEE script variables: **geometry**, **startpoints**

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE0_demo.png">

Before working with own data, comment out demo parameters!

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE0-1_demo.PNG">

### 4.2 Import user parameters

GEE script variables: **geometry**, **startpoints**

Upload area of interest and access points (previously created in QGIS or R) as shapefiles in *Assets*-tab.

1. Click "NEW".
2. Click "Shape files".
3. Select or drag & drop file. Following shapefile extensions are needed: .shp, .prj, .shx, .dbf; *(optionally rename file of asset (dotted magenta line)*.  
4. Click "UPLOAD".

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE1_upload.png">

Uploaded files will appear soon in *Assets*-tab (left window, dotted magenta line). If not click "refresh" (solid magenta line).
Copy the asset paths into script (dotted magenta lines in code editor).

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE2_scriptinput.png">

Optionally draw aoi as polygon on map. The file will automatically appear as **geometry**-variable on top of the script.
*In this case make sure all other "geometry" variables in the script (demo, asset import) are commented out.*

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE3_draw_geometry.png">
  
## 4.3 Modify other parameters

GEE script variables: **watermask**, **occ**, **buffer**, **maxPixels**

1. Set "T" (true) to mask out water areas or "F" (false) to not apply water mask. Information on global water occurrence is provided by the [Global Surface Water](https://developers.google.com/earth-engine/datasets/catalog/JRC_GSW1_4_GlobalSurfaceWater) data set.
  + Optionally modify water **occ**urrence parameter to refine water mask, but this can be done later as well (see section 4.4).  

2. Optionally change the buffer to be applied around aoi, but it should be the same buffer that was already applied in previous steps when clipping roads.

3. Only modify maximum pixels if export fails. See section 4.5.

Finally click "Run" at the top of the script (solid magenta line).

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE4_other_parameter.png">

  
## 4.4 Inspect output

After executing the script, some of the selected parameters will be displayed in the print console (right window, dotted magenta line).

*Study area*, *starting points*, *cost raster* and the *cumulative cost raster (remoteness layer)* are added to the map (hover over layer panel box in map). Optionally modify visualization parameters of single layers When clicking on "settings" of respective layer, or uncheck layer if it should not be displayed in map.

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE5_visualization.png">

<!-- ```{r} -->
<!-- knitr::include_graphics(file.path(wd, "graphics/GEE5_visualization.png")) -->
<!-- ``` -->
  
**Refine water mask**

If a water mask is applied (var **watermask = "T"**), 10 different cost raster are added to the map by default, based on different values for the probability of water occurrence (10%, 20%, ... 100%).

This way the layers can be compared with each other (check / uncheck). If the water mask should be refined, the **occ** parameter must be modified and the script must be re-executed. The water mask applied to the images ready for export are based on the specified occurrence value.

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE6_refine_watermask.png">

  
**Figure: Compare the output rasters of the remoteness analysis after applying different water masks.**

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/refine_watermask.png">

  
## 4.5 Initiate Export

Final image export needs to be initiated in 'Tasks Tab' (right window).  
1. Go to "Tasks" tab. 
  
2. Click "RUN" to initiate each export.  
  
-> Optionally modify default export parameters (dotted magenta line).  
  
  * For example, add name of aoi for better distinction.
  * If CRS should be set e.g to UTM, insert respective EPSG code.
  * A lower scale reduces output file size, higher scale than 30 meter resolution is not recommended.
  * In Google Drive, the GEE_Export folder will be automatically created if not already existing. Can change name to already existing folder.  
  
  
  
3. Click "RUN" to finally initiate the export.

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE7_initiate_export.png">


Depending of size of study area, an image export can take from minutes to days. Multiple exports can be run in parallel.


For example, if **maxPixels** parameter would have been set to 4000, an error message would occur after initiating the final export task (see Figure). In this case, the maxPixels value (see section 4.3) would have to be increased (see error message) and the script and export task would have to be executed again.

<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/GEE8_maxpixels_error.png">


