# QGIS Real World Data Tutorial

This process has often been described using Global Mapper, but I wanted to detail the equivalent steps within the open source (free) GIS app - QGIS.

The goal was to achieve an export satellite image and heightmap with the same or better quality than would be possible with Global Mapper.

The is my second revision of this process as I recently discovered that due to an update to GDAL (**2.4.0**) one of the key steps in the original tutorial using '**Clip Raster by Mask Layer**' no longer works perfectly!

So I reworked the process by getting a bit more under the hood, and using GDAL command line where possible, this also simplifies the process a little and reduced even further processing on the data.

This tutorial used the following software:

[QGIS v3.4.3](https://qgis.org/en/site/forusers/download.html)

[Terra Incognita v2.45](https://sourceforge.net/projects/terraincognita2/files/)

I have referenced Terra Incognita because i felt it was the easiest way to obtain satellite data in the oziexplorer map format - see this [tutorial](https://pmc.editing.wiki/doku.php?id=arma3:terrain:satellite-texture-terra-incognita) for help using Terra Incognita.

**First an overview of the steps required:**

## For Heightmap:

  * Load heightmap into QGIS (drag and drop)
  * Set QGIS to appropriate CRS for location of your HM
  * Create shapefile & generate perfect square using Advanced Digitizing panel
  * Obtain extents from square feature
  * Run 2 GDAL commands
    * First to set cell size, CRS and then clip to shapefile square extents
    * Second to convert to .asc file


## For Satellite image:

**For single tile export**

  * Load in just the '.map' files generated in Terra Incognita into QGIS
  * Merge rasters
  * Obtain extents from square feature
  * gdalwarp command will set cell size, CRS and then clip to shapefile square extents

**For 4 tile export (large projects 40960 and above)**

  * Load in just the '.map' files generated in Terra Incognita into QGIS
  * Merge Rasters
  * Create grid -- 2 x 2 Feature
  * Move each quarter to its own shapefile layer
  * Obtain extents from each quarter feature
  * Run gdalwarp command for each quarter to set cell size and CRS and clip to shapefile square extents

**And now detailed steps for:**


# Heightmap: 

**Drag and drop your heightmap '.asc' into QGIS**

**Set QGIS to appropriate CRS for location of your HM**

Clicking on the CRS section bottom right within QGIS -- will bring up the Project Properties for CRS

![Ross-QGIS-Tutorial-11b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-11b.png)

Select the CRS to match your real world data, in this example its ‘**WGS 84 UTM zone 20N**’ because the terrain is from the Montserrat in the Caribbean. See also [UTM projection](https://pmc.editing.wiki/doku.php?id=arma3:terrain).

This will ensure your QGIS project space works appropriately with any other data you want to add - eg road shapefiles etc. Terrain Builder will only use UTM 31N but we'll get to that later.

![Ross-QGIS-Tutorial-13b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-13b.png)

**Create shapefile & generate square feature using Advanced Digitizing panel**

Top menu -- **Layer **> **Create Layer** > **New Shapefile Layer**

Save to project folder location

Change Geometry type to ‘Polygon’

Match CRS so it is the same as that just configured in above step

![Ross-QGIS-Tutorial-14b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-14b.png)

Right click on your shapefile in layers window & select ‘**Toggle Editing**’

![Ross-QGIS-Tutorial-15b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-15b.png)

From menu -- **View** → **Toolbars** → **Advanced Digitizing Toolbar**

![Ross-QGIS-Tutorial-16.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-16.png)

Select ‘**Add Polygon Feature**’

![Ross-QGIS-Tutorial-17.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-17.png)

Select Enable advanced digitizing tools

![Ross-QGIS-Tutorial-18.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-18.png)

To create a perfect square, **left mouse click** for your **top left** starting point, move your mouse to the right a little, then press ‘**d**’ on keyboard, type in the exact width (distance) required (in this example ‘20480’) then immediately press **Enter** -- be careful not to move the mouse before pressing **Enter** or it will mess up the number. Press **left mouse click** and you will have drawn your first horizontal line which turns red. By default this tool snaps to 90 degree angles, making it easy to draw your lines. Now to draw the vertical line start moving mouse your down, press ‘d’ again and type in same figure as above, press **Enter** and another **left mouse click**, repeat the process for the last 2 lines of the square.

![Ross-QGIS-Tutorial-19b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-19b.png)

See a short video example using the **Advanced Digitizing** tool:

![AdvancedDigitizing.gif](https://www.rossedwards.co.uk/arma/tutorial/AdvancedDigitizing.gif)

After the last **left mouse click** from the step above, the tool is still waiting to plot more points, so to finalise the square do a **right mouse click**

![Ross-QGIS-Tutorial-20b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-20b.png)

This will then bring up an attributes dialogue -- just type any number and Enter

![Ross-QGIS-Tutorial-21b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-21b.png)

**Obtain extents from square feature**

Activate the toolbox if it is not already available

![Ross-QGIS-Tutorial-new-12.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-12.png)

Start typing '**vector**' in Processing Toolbox search bar, and you will see '**Vector information**'

Double click it and make sure your square **input layer** is selected - then just click **Run**

![Ross-QGIS-Tutorial-new-01.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-01.png)

You will only need to copy the **Extent** figures, which we will use within the GDAL commands in the next step

![Ross-QGIS-Tutorial-new-02.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-02.png)

**Run the first GDAL command which will - set CRS, cell size and then clip to shapefile square extents**

From menu - **Plugins** > **Python Console**

Click '**Show Editor**'

![Ross-QGIS-Tutorial-new-03.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-03.png)

The blank window on the right is where you will paste GDAL commands

Now you are ready to edit the command below to match your data - sections in bold are the parts to edit:

import os
os.system(r'''gdalwarp -t_srs EPSG:**32620** -wo SOURCE_EXTRA=1000 -tr **5.0 5.0** -srcnodata ”-9999” -r cubic -of GTiff -te **576787.687480 1841104.815839 597267.687480 1861584.815839** **D:/Arma/Heightmaps/Opentopo/output_srtm.asc D:/Arma/QGIS/Montserrat/converted.tif**''')

Some explanation of the key parameters:

**-t_srs EPSG:32620**

CRS to match where the heightmap is from in the world.

**-tr 5.0 5.0**

The desired resolution of your heightmap, match with **Cell size** set within your Mapframe properties in Terrain Builder:

![Ross-QGIS-Tutorial-new-04.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-04.png)

**-te 576787.687480 1841104.815839 597267.687480 1861584.815839**

The 4 long figures represent the **extents** of your square, and should be replaced with your own figures obtained in previous step.

Update the paths to match where you are storing your source heightmap, and where you want the output tif saved to.

Input:

**D:/Arma/Heightmaps/Opentopo/output_srtm.asc**  use double quotes around your input path if it contains any blanks

Output:

**D:/Arma/QGIS/Montserrat/converted.tif**

**Run second GDAL command - converting tif from above step to a Terrain Builder friendly .asc**

Remember to change the paths in below command to match where your files are located

import os
os.system(r'''gdal_translate -of AAIGrid D:/Arma/QGIS/Montserrat/converted.tif D:/Arma/QGIS/Montserrat/final.asc''')


# Satellite image:

==== For single tile export ====

**Load in just the '.map' files generated from Terra incognita into QGIS**

![Ross-QGIS-Tutorial-1b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-1b.png)

**Merge Rasters**

Top Menu -** Raster **→ **Miscellaneous **→ **Merge**

Select all your .map files as input layers

![Ross-QGIS-Tutorial-3b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-3b.png)

change **Output data type** to ‘Byte’ as the default is ‘Float32’.

Select ‘**Save to File**’ under Merged -- as the default is to a temporary file.

![Ross-QGIS-Tutorial-2b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-2b.png)

**Obtain extents from square feature**

You should already have this from working on heightmap above   **576787.687480 1841104.815839 597267.687480 1861584.815839**

**Run GDAL command**

import os
os.system(r'''gdalwarp -t_srs EPSG:32620 -r cubic -wo SOURCE_EXTRA=1000 -tr 1.0 1.0 -r cubic -of BMP -te 576787.687480 1841104.815839 597267.687480 1861584.815839 D:/Arma/QGIS/Montserrat/merged.tif D:/Arma/QGIS/Montserrat/mont.bmp''')

Some explanation of the key parameters:

**-t_srs EPSG:32620**\\
Sets the CRS

**-tr 1.0 1.0**\\
Sets the resolution of your satellite image to 1mtr per pixel - which is recommended. Make sure it matches your settings within Terrain Builder.

![Ross-QGIS-Tutorial-new-05.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-05.png)

**-te 576787.687480 1841104.815839 597267.687480 1861584.815839**\\
Clips to shapefile square extents


==== For 4 tile export ====

Detailing additional steps required for a 4 tile export -- which would suit those terrains that are 40960px x 40960px and higher.

**Load in just the .map files into QGIS**

**Merge rasters - same as above (single tile export)**

**Create grid -- 2 x 2 Feature**

You will need to have created the **square shapefile** first -- see above steps

Top Menu -- **View** → **Panels** → **Processing Toolbox**

Processing Toolbox Menu -- **Vector Creation** > **Create Grid**

Set **Grid type** to ‘**Rectangle (polygon)**’ & the **Horizontal & Vertical** to half the distance of your main square

Set it to save new grid feature to .shp file

![Ross-QGIS-Tutorial-26b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-26b.png)

After clicking '**Use Layer Extent**' Select the shapefile layer for your square

![Ross-QGIS-Tutorial-new-13.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-13.png)

After clicking **Run**, your original square will then be Covered with a new grid of 4 tiles -- perfect quarters of your original square

![Ross-QGIS-Tutorial-30b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-30b.png)

**Move each quarter to its own shapefile layer**

Because you will need to export each quarter separately for use in Terrain Builder, we will move each quarter onto its own shapefile layer.

Click **Select Features** -

![Ross-QGIS-Tutorial-31.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-31.png)

And select the first quarter:

![Ross-QGIS-Tutorial-32b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-32b.png)

Top Menu -- **Edit** > **Copy Features**

Then… **Edit** → **Paste Features As **→ **New Vector Layer**

Save shapefile layer to something obvious

I’ve used ‘**TL**’ for top left.

![Ross-QGIS-Tutorial-33b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-33b.png)

Repeat for the remaining quarters and you’ll have something like below, ‘Grid’ can be deleted.

![Ross-QGIS-Tutorial-34b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-34b.png)

You will then have 4 tiles which we can use to gather each quarters extents

![Ross-QGIS-Tutorial-35b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-35b.png)

**Obtain extents from each quarter feature**

![Ross-QGIS-Tutorial-new-06.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-06.png)

Copy extents to clipboard

![Ross-QGIS-Tutorial-new-07.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-07.png)

And list them out for safe keeping -

TL -  Extent: (576787.687480, 1851344.815839) - (587027.687480, 1861584.815839)

TR -  Extent: (587027.687480, 1851344.815839) - (597267.687480, 1861584.815839)

BL -  Extent: (576787.687480, 1841104.815839) - (587027.687480, 1851344.815839)

BR - Extent: (587027.687480, 1841104.815839) - (597267.687480, 1851344.815839)

**Run GDAL command for each quarter to set cell size and CRS and clip to shapefile square extents**

import os
os.system(r'''gdalwarp -t_srs EPSG:32620 -r cubic -wo SOURCE_EXTRA=1000 -tr 1.0 1.0 -r cubic -of BMP -te 576787.687480 1841104.815839 597267.687480 1861584.815839 D:/Arma/QGIS/Montserrat/merged.tif D:/Arma/QGIS/Montserrat/TR.bmp''')


# Loading your assets into Terrain Builder


**Heightmap (.asc) edits**

Open your asc and change the following:

![Ross-QGIS-Tutorial-new-14.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-14.png)

Before loading the asc file into Terrain Builder, delete the .prj file. Otherwise it will not import into TB.

![Ross-QGIS-Tutorial-new-09.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-09.png)

**Satellite image**


Before loading the satellite image into Terrrain Builder delete the .bmp.aux file

![Ross-QGIS-Tutorial-new-08.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-08.png)


# Managing the project assets within QGIS


Save your QGIS project so that you can return to export further data at a later date.


Layers can be tidied up to hold the essentials ready for the next time you want to re-run GDAL commands, or export anything else like road shapefiles or mask layers etc.


![Ross-QGIS-Tutorial-new-11.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-new-11.png)

**Coming Soon:**
  * Road shapefile creation
  * Assisted image classification - for creation of mask
  * 3d preview using Qgis2threejs

