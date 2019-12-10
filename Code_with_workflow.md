

###                                                                                 **Code with workflow**

 

Open GRASS 7.6 through OSGeo

`pdal translate --input=study_area_lidar_data.las --writers.las.minor_version=2 --output=study_area_lidar_data_v12.las`

Open normal GRASS GIS

\# Coordinate system is 2264. Convert to 3358

`lasinfo study_area_lidar_data_v12.las`

`las2las --a_srs=EPSG:2264 --t_srs=EPSG:3358 -i study_area_lidar_data_v12.las -o study_area_lidar_data_v12_3358.las`

\# Removing noise from LAS file in **lasnoise tool.**

`lasnoise -cpu64 -i "C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Study_area_Lidar_data\study_area_lidar_data_v12_3358.las" -step_xy 4 -step_z 1 -remove_noise -odir "C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Study_area_Lidar_data" -odix "_denoised" -olas`

\#Computing DSM

`r.in.lidar -e -n input=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and` 

`Simulations\Final_project\Data\Study_area_Lidar_data\study_area_lidar_data_v12_3358_denoised.las output=dsm_1m method=max resolution=1 zrange=0,275 –overwrite`

\# Gap filling

`r.fill.stats -k input=dsm_50cm@PERMANENT output=dsm_50cm_filled distance=3 power=2.0 cells=`8

Ran the above code 6 times (varying inputs: dsm_50cm _filled1, dsm_50cm _filled2, dsm_50cm _filled3, dsm_50cm _filled4, dsm_50cm _filled5 ) to finally obtain dsm_50cm_filled6.

\# r.sun

Mode 2

`r.sun elevation=dsm_50cm_filled6@PERMANENT glob_rad=jun21_total_irradiance_50cm insol_time=jun21_insolation_50cm day=172 nprocs=6`

\# importing parking lots shapefile into GRASS

`v.in.ogr input=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Parking_Areas\Parking_Areas_3358_clipped.shp output=parking_lots`

\# Extracting values from raster

`v.rast.stats map=parking_lots@PERMANENT raster=jun21_total_irradiance@PERMANENT column_prefix=Total_jun21`

This will create many unnecessary columns than necessary.

\# changing color of vector file

`v.colors map=parking_lots@PERMANENT use=attr column=Total_jun21_sum color=bcyr`

For outline of polygons only: double click on parking_lots@PERMANENT, (d.vect pops up). Selection --> layer =-1. Colors--> Area fill color --> Transparent

\# Exporting as a Geopackage instead of shapefile

`v.out.ogr -c input=parking_lots@PERMANENT output=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Outputs\parking_lots_solar_jun21.gpkg format=GPKG`

`r.in.lidar input=soccer_field_v_12_3358.las output=count_1 method=max -e -n resolution=1`

\# Fill data gaps

`r.fill.stats input=count_1@PERMANENT output=count1_filled distance=3 mode=wmean power=2.0 –overwrite`

\# Gaps not filled entirely. Repeat again on the newly filled raster

`r.fill.stats input=count1_filled@PERMANENT output=count1_filled2@PERMANENT distance=3 mode=wmean power=2.0 cells=8`

\# r.fill and r.stats reducing quality of outputs.

\# Trying with v.in.lidar

25Mar19

\# Vegetation only from Lidar

`r.in.lidar input=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Study_area_Lidar_data\study_area_lidar_data_v12_3358_denoised.las output=Vegetation_3_4_5 method=max class_filter=3,4,5 resolution=1 –o`

​                           





​           Tree Mask = if (DSM- (DEM + Building heights) >2.5 ? 1      : 0            

\# Calculating the DEM 

`r.in.lidar --overwrite input=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\Study_area_Lidar_data\study_area_lidar_data_v12_3358_denoised.las output=Ground_2_13_14 method=min resolution=0.5 class_filter=2,13,14`

This was then smoothed many times using r.fill.stats. Ground_2_13_14_filled12 is the final filled DEM.(Please note that this file still have No data values in area belonged by buildings(some building area got filled while some didn’t). The unfilled building areas (with No Data values) were filled with a value 1. This could have been avoided if the r.fill.stats could have run around 100 times, which is cumbersome.)



`r.mapcalc --overwrite expression=DSM_minus_DEM = dsm_50cm_filled6@PERMANENT - Ground_2_13_14_filled12@PERMANENT >= 2? dsm_50cm_filled6@PERMANENT - Ground_2_13_14_filled12@PERMANENT:0`

`r.mapcalc expression=Tree_and_portion_of_buildings=if(DSM_minus_DEM>2,1,0)`

 

 

02Apr19

\# Creating building mask

`v.to.rast input=Raleigh_Buildings_clip@PERMANENT type=area output=Buildings_mask use=val`

`r.null map=Buildings_mask@PERMANENT null=0`

\# Creating a DEM with building heights added to it

`r.mapcalc expression=dsm_buildings_only = dsm_50cm_filled6@PERMANENT * Buildings_mask@PERMANENT`

`r.mapcalc --overwrite expression=DEM_with_building_heights = Ground_2_13_14_filled12+dsm_buildings_only`

\# Creating tree mask

`r.mapcalc expression=Trees = (dsm_50cm_filled6@PERMANENT-DEM_with_building_heights@PERMANENT)>2?1:0`

\# Creating inverted tree mask

\# Removing tree-values from jun21_total_irradiance_50cm by multiplying with mask

`r.mapcalc expression=jun21_total_irradiance_50cm_trees_removed = jun21_total_irradiance_50cm@PERMANENT * Trees_inverted@PERMANENT`

\# Substituting a value of 1216 (the least value for a pixel under 24 hours of shade) for tree pixels

`r.mapcalc expression=jun21_total_irradiance_50cm_trees_adjusted = jun21_total_irradiance_50cm_trees_removed@PERMANENT==0?1216:jun21_total_irradiance_50cm_trees_removed`

\# Extracting values from raster

`v.rast.stats map=Parking_Areas_3358_clipped@PERMANENT raster=jun21_total_irradiance_50cm_trees_adjusted@PERMANENT column_prefix=Total_irr_Jun21_ method=minimum,maximum,average,sum`



 

Mosaicing in GRASS

\# import all the 6 GI rasters into GRASS one by one

`r.import input=C:\Users\vishn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Simulations\Final_project\Data\6_inch_resolution_Imagery\20170403_3358_GI.tif output=20170403_3358_GI`

\# set the computational region to an extent encompassing all the 4 rasters

`g.region raster=20170301_3358_GI@PERMANENT,20170302_3358_GI@PERMANENT,20170403_3358_GI@PERMANENT,20170404_3358_GI@PERMANENT,20171301_3358_GI@PERMANENT,20171403_3358_GI@PERMANENT`

\#mosaicking using r.patch command

`r.patch r.patch --overwrite input=20170301_3358_GI@PERMANENT,20170302_3358_GI@PERMANENT,20170403_3358_GI@PERMANENT,20170404_3358_GI@PERMANENT,20171301_3358_GI@PERMANENT,20171403_3358_GI@PERMANENT output=GI_patched`

\# Set the computational region with extent = tree mask and resolution = 0.5

`g.region raster=Trees@PERMANENT res=0.5`

\# Create GI raster of trees alone using tree mask.

`r.mapcalc expression=GI_patched_tree_mask = GI_patched@PERMANENT * Trees@PERMANENT`

\# Create GI raster of evergreen trees. 0.375 is the GI threshold value for evergreen trees

`r.mapcalc expression=GI_patched_tree_mask_evergreen = GI_patched_tree_mask@PERMANENT >=0.375`

\# Create GI raster of deciduous trees. 

`r.mapcalc --overwrite expression=GI_patched_tree_mask_deciduous = GI_patched_tree_mask@PERMANENT <0.375 && GI_patched_tree_mask@PERMANENT !=0`

\# Extract height of deciduous trees from DSM

`r.mapcalc expression= heights_of_deciduous_only = dsm_50cm_filled6@PERMANENT * GI_patched_tree_mask_deciduous@PERMANENT`

\# DEM+deciduous

`r.mapcalc expression=DEM_plus_deciduous = if(heights_of_deciduous_only @PERMANENT>0, heights_of_deciduous_only @PERMANENT, Ground_2_13_14_filled_iteration50@PERMANENT)`

\# r.sun on DEM+deciduous

\# Extract height of evergreen trees from DSM

`r.mapcalc expression=heights_of_evergreen_only = dsm_50cm_filled6@PERMANENT * GI_patched_tree_mask_evergreen@PERMANENT`

\# DEM+evergreen

`r.mapcalc expression=DEM_plus_evergreen = if( heights_of_evergreen_only@PERMANENT >0, heights_of_evergreen_only@PERMANENT, Ground_2_13_14_filled_iteration50@PERMANENT)`

\# r.sun on DEM+evergreen

`r.sun elevation=DEM_plus_evergreen@PERMANENT glob_rad=Evergreen_total_irradiance_01jan insol_time=Evergreen_insolation_01jan day=1 nprocs=6`

 

 

\# creating a mask (or extracting) total irradiance of pixels that falls under deciduous trees at any time of the day. The rest of the pixels are assigned zero. 

`r.mapcalc expression=brightest_pixels_removed_from_deciduous_shade = if( Deciduous_total_irradiance_01jan@PERMANENT >= 2989 ,0,Deciduous_total_irradiance_01jan@PERMANENT)`

\# Adjusting by a factor of 0.66

`r.mapcalc expression= brightest_pixels_removed_from_deciduous_shade_adjusted_by_066 = brightest_pixels_removed_from_deciduous_shade @PERMANENT + 0.66*(3119- brightest_pixels_removed_from_deciduous_shade @PERMANENT)`

\# Substituting zero values with r.sun(DEM) value. Zero values have become 2058.54 now.

`r.mapcalc --overwrite expression=Adjusted_total_irradiance_by_dem_plus_deciduous_01jan = if( brightest_pixels_removed_from_deciduous_shade_adjusted_by_066@PERMANENT ==2058.54, DEM_total_irradiance_01jan@PERMANENT , brightest_pixels_removed_from_deciduous_shade_adjusted_by_066@PERMANENT )`

\## Taking compliment of evergreen Tree mask.

`r.mapcalc expression=GI_patched_tree_mask_evergreen_compliment = if(GI_patched_tree_mask_evergreen@PERMANENT==0,1,0 )`

\#Tree-pixel removal for evergreen

`r.mapcalc expression=Evergreen_total_irradiance_01jan_zero_beneath_trees = Evergreen_total_irradiance_01jan@PERMANENT* GI_patched_tree_mask_evergreen_compliment@PERMANENT`

`r.mapcalc expression=Evergreen_total_irradiance_01jan_zero_beneath_trees_adjusted = if(Evergreen_total_irradiance_01jan_zero_beneath_trees@PERMANENT==0,678,Evergreen_total_irradiance_01jan_zero_beneath_trees)`

\## Taking compliment of evergreen Tree mask.

`r.mapcalc expression=GI_patched_tree_mask_deciduous_compliment = if(GI_patched_tree_mask_deciduous@PERMANENT==0,1,0)`

  

\#Tree-pixel removal for adjusted deciduous and substituting zero values with 2280 [New pixel value = min + 0.66 *(max-min) ] Here min = 678 and max = 3105

`r.mapcalc expression=Adjusted_total_irradiance_by_dem_plus_deciduous_01jan_zero_beneath_trees = Adjusted_total_irradiance_by_dem_plus_deciduous_01jan@PERMANENT* GI_patched_tree_mask_deciduous_compliment@PERMANENT`

`r.mapcalc expression=Adjusted_total_irradiance_by_dem_plus_deciduous_01jan_zero_beneath_trees_adjusted = if(Adjusted_total_irradiance_by_dem_plus_deciduous_01jan_zero_beneath_trees@PERMANENT==0,2280,Adjusted_total_irradiance_by_dem_plus_deciduous_01jan_zero_beneath_trees)`

 

 

\# Finding the minimum among the three

`r.mapcalc --overwrite expression=Total_irradiance_minm_among_all_three_01jan = min( Adjusted_total_irradiance_by_dem_plus_deciduous_01jan_zero_beneath_trees_adjusted@PERMANENT , Evergreen_total_irradiance_01jan_zero_beneath_trees_adjusted@PERMANENT , DEM_plus_buildings_total_irradiance_01jan@PERMANENT )`

\# Extracting values from raster

`v.rast.stats map=Parking_Areas_3358_clipped@PERMANENT raster=Total_irradiance_minm_among_all_three_01jan@PERMANENT column_prefix=Jan method=minimum,maximum,average`

\# Exporting as Geopackage

`v.out.ogr input=Parking_Areas_3358_clipped@PERMANENT output=C:\Users\vvivekn\Google Drive\NCSU_Courseware\GIS_714_Geocomputing and Smulations\Final_project\Outputs\Parking_Areas_solar_irradiance_Jun_and_Jan.gpkg format=GPKG`

\# Extracting values from raster (for Roads)

`v.rast.stats map=Roads_without_median`



 
