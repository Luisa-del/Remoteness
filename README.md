# Remoteness analysis

This is a step-by-step tutorial on how to perform a remoteness analysis for user-defined areas. The tutorial is split into two parts:
1. [Prepare input data using QGIS (optionally R)](#1-Prepare-input-data) ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_qgis.png) 
![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_r.png)
2. [Perform remoteness analysis in Google Earth Engine (GEE)](#2-perform-remoteness-analysis) ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_gee.png) 

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

3. **Choose software to download & prepare osm data (size dependent!).** Simplest is to use the **"QuickOSM-plugin"** from QGIS for *smaller* areas like National Parks or province borders. Here you can download the data already at the size of your buffered AOI and filter it by drivable road categories -------------------(see section ...). If the study area is *too large* for QuickOSM (like a whole country), either use smaller tiles in QuickOSM, or download and prepare the data in R. Here you can download the data on a country level, filter it by drivable road category and then optionally further clip or merge to your buffered AOI -----------------------------(more info see section ...). You need to try what's best for you. The larger the study area and amount of data, the longer the processing!

4. **If possible (!) verify osm roads.** OpenStreetMap is a community-based, freely available, editable map service and provides free and open source geographical data sets of natural or man made features. Given that it is edited mainly by volunteers with different mapping skills, the completeness and quality of its annotations are heterogeneous across different geographical locations, and updates are more regularly in urban areas than in rural areas. If possible, verify whether the selected road data represents the road network of the study area as good as possible. 

5. **Convert osm drivable roads to starting points.** Use the **"Points along geometry"-tool** from QGIS to convert drivable road lines to equally spaced points. Sufficient spacing should be 50 or 100 meters.

6. **Use three softwares to perform analysis (R, QGIS, Google Earth Engine).** R and QGIS must be used to prepare the osm data and create the starting points. The final remoteness analysis will be performed in Google Earth Engine (GEE).
    + Filtering for road categories is quicker and easier to perform in R (especially with larger data sets). 
    + The conversion of road lines to points can only be performed in QGIS. 
    + All other processing steps can be performed in either R or QGIS, depending on the user's preference.

Most of the steps in this tutorial are performed in R and the code is provided. The steps performed in QGIS or GEE are described with screenshots.
8. **Carefully select data for GEE upload.** Only upload starting points and study area (not buffered study area!). Apply the same buffer in the GEE script that was used to create the starting points (see section 4).
9. **GPS points available?** In case you want to use other starting point for your analysis (e.g GPS points along the road network, villages,...) you only need to upload your study area and the starting point dataset to GEE. Consider a buffer!


## 1. Prepare input data




## 2. Perform remoteness analysis




