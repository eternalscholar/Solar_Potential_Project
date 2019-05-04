### Problems Faced

1. No support for Lidar version1.4 in GRASS yet. It needs to be converted to 1.2. 

2. pdal translate works only from GRASS 7.6 opened through OSGeo. GRASS 7.6 installed on desktop can't successfully use pdal.

3. To create a tree mask ( or a raster with trees only(tree heights only)), we could have used the “r.in.Lidar” with classification categories of vegetation only. But, it is very difficult to fill holes within the vegetation raster without growing the canopy circumference. No ready made algorithms exists in GRASS/ArcGIS/R. Tried the methods given here <https://stackoverflow.com/questions/41186218/fill-raster-holes-in-r-or-grass-gis>. But R has memory constraints, which means that the study area needs to be split into very small pieces, which won’t be good. 

4. Class categories incorrectly assigned in Lidar data

5. NDVI from NAIP will not work out in our study because NAIP doesn’t seem to be orthorectified. So the tree mask from Lidar data may not exactly overlap with tree top from NAIP.



**6. Creating GI and problems faced**

1) Raster calculator runs in ArcGIS, but no output. So QGIS was the next best choice. All the 6  rasters where exported to Tiff format from SID format in ArcGIS and then reprojected to 3358. (Choose the option “No” if ArcGIS asks to alter pixel depth)
 2) QGIS needs to be closed after running raster calculator once. In total 6 raster calculator operations were needed to compute GI for 6 high resolution rasters.
 3) No direct tools for mosaicking in ArcGIS. Raster to Mosaic didn’t work. Raster calculator can be used where the component rasters can be added together. Needs to set extent to the extent of all the 6 rasters.

