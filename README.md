# Remoteness analysis

## Overview

This document provides a step-by-step tutorial on how to perform a remoteness analysis for a user-defined area using R, QGIS and Google Earth Engine. Remoteness indicates the distance to roads, but takes into account the cost to walk through terrain. It can serve as anthropogenic factor in wildlife studies, where it has been shown to be an important driver of species occurrence.
  
* Costs are calculated from a hiking function that considers slope. The function is also defined to take metabolic costs into account, because it has been shown that humans are typically not following least-time routes, especially in mountainous areas.
  
* Drivable roads can serve as starting points from where the costs are then added up (cumulative costs) in order to assess remoteness. This can be points along drivable roads from OpenStreetMap (osm), or GPS points.

* The drivable roads from where the costs are added up to determine remoteness should be within the study area, but furthermore also within a certain buffer area around the study area. This ensures that roads in the vicinity of the study area are also included in the calculation of remoteness and avoids distorted remoteness values at the border of the study area.

* Some areas like water bodies are inaccessible and need to be masked out prior to the analysis.

![](".png")

*Figure: Remoteness layer for province of Kâmpóng Thum (Cambodia), using osm drivable roads and GLobal Surface Water mask.*
<img src="https://github.com/Luisa-del/Remoteness/blob/main/img/remoteness.png" width = "700">

1. All input data can be prepared in QGIS, if the study area is too large fpr QuickOSM plugin we need to use both R and QGIS. ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_qgis.png) ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_r.png).

2. Remoteness analysis will be performed in Google Earth Engine. ![](D:/Dateien/Uni/Eagle_Master/Hiwijob_IZW/Remoteness_tutorial/graphics/logo_gee.png) 

![](".png")





