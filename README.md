# Arma3_QGIS

I will be running through the steps to export satellite images from QGIS ready for use in Terrain Builder

Software required:
Terra Incognita   https://sourceforge.net/projects/terraincognita2/
QGIS 3.2.0   https://qgis.org/en/site/forusers/download.html

Brief overview of steps:

For single tile export 

•	Load in just the .map files into QGIS

•	Merge Rasters

•	Align raster to 1mtr cell size and re-project to appropriate CRS for its geo location

•	Change the ‘Project Coordinate Reference System’ (CRS) to match CRS in previous step

•	Create shapefile layer

•	Create square Feature using Advanced Digitizing panel

•	Export sat image ready for Terrain Builder using ‘clip raster by mask layer’


For 4 tile export 

•	Load in just the .map files into QGIS 

•	Merge Rasters

•	Align raster to 1mtr cell size and re-project to appropriate CRS for its geo location

•	Change the ‘Project Coordinate Reference System’ (CRS) to match CRS in previous step

•	Create shapefile layer

•	Create grid – 2 x 2 Feature 

•	Move each quarter to its own shapefile layer

•	Export 4 sat images ready for Terrain Builder using ‘clip raster by mask layer’


I shall primarily focus on the steps for generating a single tile export, but will explain additional steps required for a 4 tile export at the end – which would suit those terrains that are 40960px x 40960px and higher.
