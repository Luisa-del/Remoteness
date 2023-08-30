# Remoteness analysis

This document provides a step-by-step tutorial on how to perform a remoteness analysis for a user-defined areas.

## Overview

* This tutorial is split into two parts:
    1. Prepare input data using QGIS (optionally R) ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_qgis.png) 
![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_r.png)
    2. Perform remoteness analysis in Google Earth Engine (GEE) ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_gee.png) 

* Remoteness indicates the distance to roads, but takes into account the cost to walk through terrain. It can serve as anthropogenic factor in wildlife studies, where it has been shown to be an important driver of species occurrence.
  
* Costs are calculated from a hiking function that considers slope. The function is also defined to take metabolic costs into account, because it has been shown that humans are typically not following least-time routes, especially in mountainous areas.
  
* Drivable roads can serve as starting points from where the costs are then added up (cumulative costs) in order to assess remoteness. This can be points along drivable roads from OpenStreetMap (osm), or GPS points.

* The drivable roads should cover a certain buffer area around the study area. This ensures that roads in the vicinity of the study area are also included in the calculation of remoteness and avoids distorted remoteness values at the border of the study area.

* Some areas like water bodies or other inaccessible areas affect accessibility and can be masked out prior to the analysis.

![](".png")

*Figure: Remoteness layer for province of Kâmpóng Thum (Cambodia), using osm drivable roads and GLobal Surface Water mask.*
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/remoteness.png" width = "700">


![](".png")

## 1. Useful information when performing remoteness analysis 

1. **Apply sufficient buffer around study area.** A buffer area of 10 km around the study area was considered large enough to ensure that roads that have a relevant influence on the remoteness values at the border of the Aoi are included in the analysis. The buffer value can be modified by the user (see section 2.1).
2. **Select country for osm download.** [GADM](https://gadm.org/download_country.html) provides free and easy accessible country data at different subdivision levels (see section 2.1 & 2.2). 
3. **Choose method for downloading OpenStreetMap data.** There are different ways to download and prepare osm data (more info see section 2.3).
4. **Carefully prepare osm roads.** The conversion of the osm roads to starting points can vary in effort / time depending on the size of the downloaded data set. It requires the selection of drivable road categories, clipping the data set to the buffered aoi, and converting the line features to points along these lines at a regular spacing (see section 3.1 - 3.5).
5. **Check osm roads before interpolation.** If possible (!), verify whether the selected road dataset represents the road network of the study area as good as possible.  
OpenStreetMap is a community-based, freely available, editable map service and provides free and open source geographical data sets of natural or man made features. Given that it is edited mainly by volunteers with different mapping skills, the completeness and quality of its annotations are heterogeneous across different geographical locations, and updates are more regularly in urban areas than in rural areas. 
6. **Use three softwares to perform analysis (R, QGIS, Google Earth Engine).** R and QGIS must be used to prepare the osm data and create the starting points. The final remoteness analysis will be performed in Google Earth Engine (GEE).
    + Filtering for road categories is quicker and easier to perform in R (especially with larger data sets). 
    + The conversion of road lines to points can only be performed in QGIS. 
    + All other processing steps can be performed in either R or QGIS, depending on the user's preference.  
Most of the steps in this tutorial are performed in R and the code is provided. The steps performed in QGIS or GEE are described with screenshots.
7. **Carefully select data for GEE upload.** Only upload starting points and study area (not buffered study area!). Apply the same buffer in the GEE script that was used to create the starting points (see section 4).
8. **GPS points available?** In case you want to use other starting point for your analysis (e.g GPS points along the road network, villages,...) you only need to upload your study area and the starting point dataset to GEE. Consider a buffer!



