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

  * Load in just the '.map' files generated from Terra Incognita into QGIS
  * Merge rasters
  * Obtain extents from square feature
  * gdalwarp command will set cell size, CRS and then clip to shapefile square extents

**For 4 tile export (large projects 40960 and above)**

  * Load in just the '.map' files generated from Terra Incognita into QGIS
  * Merge Rasters
  * Create grid -- 2 x 2 Feature
  * Move each quarter to its own shapefile layer
  * Obtain extents from each quarter feature
  * Run gdalwarp command for each quarter to set cell size and CRS and clip to shapefile square extents

**And now detailed steps for:**


# Heightmap: 

**Drag and drop your heightmap '.asc' into QGIS**

This tutorial covers single raster heightmap as per the kind you will find on opentopo [here](http://opentopo.sdsc.edu/raster?opentopoID=OTSRTM.082015.4326.1), I will detail steps for heightmaps spread across muliptle rasters in another update. as they will require additional steps likely including Merge / Build Virtual Raster commands.

**Set QGIS to appropriate CRS for location of your HM**

Clicking on the CRS section bottom right within QGIS -- will bring up the Project Properties for CRS

![Ross-QGIS-Tutorial-11b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-11b.png)

Select the CRS to match your real world data, in this example its ‘**WGS 84 UTM zone 20N**’ because the terrain is from the Montserrat in the Caribbean. See also [UTM projection](https://pmc.editing.wiki/doku.php?id=arma3:terrain).

This will ensure your QGIS project space works appropriately with any other data you want to add - eg road shapefiles etc. Terrain Builder will only use UTM 31N but we'll get to that later.

![Ross-QGIS-Tutorial-13b.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-13b.png)

**Create shapefile & generate square feature using Advanced Digitizing panel**

Top menu -- **Layer** > **Create Layer** > **New Shapefile Layer**

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

![Ross-QGIS-Tutorial-21.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-21.png)

**Obtain extents from square feature**

Activate the toolbox if it is not already available

![Ross-QGIS-Tutorial-new-12.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-12.png)

Start typing '**vector**' in Processing Toolbox search bar, and you will see '**Vector information**'

Double click it and make sure your square **input layer** is selected - then just click **Run**

![Ross-QGIS-Tutorial-new-01.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-01.png)

You will only need to copy the **Extent** figures, which we will use within the GDAL commands in the next step

![Ross-QGIS-Tutorial-new-02.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-02.png)

**Run the first GDAL command which will - set CRS, cell size and then clip to shapefile square extents**

From menu - **Plugins** > **Python Console**

Click '**Show Editor**'

![Ross-QGIS-Tutorial-new-03.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-03.png)

The blank window on the right is where you will paste GDAL commands

Now you are ready to edit the command below to match your data - sections in bold are the parts to edit:

import os
os.system(r'''gdalwarp -t_srs EPSG:**32620** -wo SOURCE_EXTRA=1000 -tr **5.0 5.0** -srcnodata ”-9999” -r cubic -of GTiff -te **576787.687480 1841104.815839 597267.687480 1861584.815839** **D:/Arma/Heightmaps/Opentopo/output_srtm.asc D:/Arma/QGIS/Montserrat/converted.tif**''')

Some explanation of the key parameters:

**-t_srs EPSG:32620**

CRS to match where the heightmap is from in the world.

**-tr 5.0 5.0**

The desired resolution of your heightmap, match with **Cell size** set within your Mapframe properties in Terrain Builder:

![Ross-QGIS-Tutorial-new-04.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-04.png)

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

## For single tile export

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
os.system(r'''gdalwarp -t_srs EPSG:**32620** -r cubic -wo SOURCE_EXTRA=1000 -tr **1.0 1.0** -r cubic -of BMP -te **576787.687480 1841104.815839 597267.687480 1861584.815839** **D:/Arma/QGIS/Montserrat/merged.tif D:/Arma/QGIS/Montserrat/mont.bmp**''')

Some explanation of the key parameters:

**-t_srs EPSG:32620**

Sets the CRS

**-tr 1.0 1.0**

Sets the resolution of your satellite image to 1mtr per pixel - which is recommended. Make sure it matches your settings within Terrain Builder.

![Ross-QGIS-Tutorial-new-05.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-05.png)

**-te 576787.687480 1841104.815839 597267.687480 1861584.815839**

Clips to shapefile square extents


## For 4 tile export

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

![Ross-QGIS-Tutorial-new-13.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-13.png)

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

![Ross-QGIS-Tutorial-new-06.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-06.png)

Copy extents to clipboard

![Ross-QGIS-Tutorial-new-07.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-07.png)

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

![Ross-QGIS-Tutorial-new-14.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-14.png)

Before loading the asc file into Terrain Builder, delete the .prj file. Otherwise it will not import into TB.

![Ross-QGIS-Tutorial-new-09.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-09.png)

**Satellite image**


Before loading the satellite image into Terrrain Builder delete the .bmp.aux file

![Ross-QGIS-Tutorial-new-08.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-08.png)


# Managing the project assets within QGIS


Save your QGIS project so that you can return to export further data at a later date.


Layers can be tidied up to hold the essentials ready for the next time you want to re-run GDAL commands, or export anything else like road shapefiles or mask layers etc.


![Ross-QGIS-Tutorial-new-11.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-11.png)

# Preparing Road Shapefile

**Load into QGIS (drag and drop) road shapefile**

![Ross-QGIS-Tutorial-37.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-37.png)

**Clip vector to square shape**
Top Menu - **Vector** > **Geoprocessing Tools** > **Clip**

![Ross-QGIS-Tutorial-38.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-38.png)

Save features from clip to new shapefile -  selecting correct **CRS UTM-20**\\

Top Menu - **Layer** > **Save As**

![Ross-QGIS-Tutorial-39.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-39.png)

**Set road shapefile to CRS - 31N**

RMB on road shapefile > **Set CRS** > **Set Layer CRS**

![Ross-QGIS-Tutorial-40.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-40.png)

![Ross-QGIS-Tutorial-41.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-41.png)

**Set QGIS project CRS to UTM- 31N (bottom right corner):**

![Ross-QGIS-Tutorial-42.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-42.png)

**Collect extents from your square shapefile:**

![Ross-QGIS-Tutorial-43.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-43.png)

Extent: (**576787.687480**, **1841104.815839**) - (597267.687480, 1861584.815839)

The first two values will be used in the v.transform step below.


**v.transform on road shapefile using calculated extents from square shapefile**

Your heightmap asc must adhere to Terrain Builders required values of **easting 200000** and **northing 0** 
Your road shapefile also needs to line up to the same values - we will use **v.transform** to achieve this

So taking the extent values we grabbed in previous step -

We will do the following calculations to bring these values to (**200000** and **0**):

**Easting:  576787.687480  - 376787.68748 = 200000**

**Northing: 1841104.815839 - 1841104.815839 = 0**

**-376787.68748**

**-1841104.815839**


Having figured out the correct figures plug them into the **v.transform** process

![Ross-QGIS-Tutorial-44.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-44.png)

Drag and Drop into QGIS your heightmap asc - the one you are able to load into Terrain builder - already adjusted to (**200000**, **0**)

Set CRS on above asc to **UTM - 31N**

Right click on asc in **Layers** > **Zoom to Layer**

In **Layers** panel drag the asc below your **transformed road shapefile** - so you can see the roads layered above the asc

![Ross-QGIS-Tutorial-45.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-45.png)

Your roads should now be perfectly aligned above your heightmap

![Ross-QGIS-Tutorial-46.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-46.png)

## Setting ID and ORDER fields

RMB on transformed shapefile layer > Toggle **Editing**

RMB on transformed shapefile layer  > Open **Attribute Table**


Click **Delete field** - select all fields and press **OK**

![Ross-QGIS-Tutorial-47.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-47.png)

New Field - Name '**ID**' - length **0**

![Ross-QGIS-Tutorial-48.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-48.png)

New Field - Name '**ORDER**' - length 0

![Ross-QGIS-Tutorial-49.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-49.png)

Toggle - **Multi Edit mode**

![Ross-QGIS-Tutorial-50.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-50.png)

ID - Enter **0** - click **Update All**

![Ross-QGIS-Tutorial-51.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-51.png)

Select ORDER from drop down - Enter **1** - click **Update all**

![Ross-QGIS-Tutorial-52.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-52.png)

Click - **Switch to table view**

![Ross-QGIS-Tutorial-53.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-53.png)

Click **Save**

![Ross-QGIS-Tutorial-54.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-54.png)

RMB on transformed road shapefile - **Toggle Editing** to save the changes 


**Switching to Terrain Builder -**
 

**Load your road shapefile**

Top menu - **File** > **Import** > **Shapes**

Click **OK**

![Ross-QGIS-Tutorial-55.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-55.png)

Perfectly overlaid within **Terrain builder:**

![Ross-QGIS-Tutorial-56.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-56.png)

# 3D Preview using Qgis2threejs

If you want to see an instant 3d preview of your terrain within QGIS you can with the amazing plugin **Qgis2threejs**

First install it:

Top menu > **Plugins > Manage and Install Plugins**

![Ross-QGIS-Tutorial-57.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-57.png)


RMB on the Merged layer and select **Zoom to Layer**


Then select both the **Merged** raster, and the original source heightmap - deselect all other layers

![Ross-QGIS-Tutorial-58.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-58.png)


Start the Qgis2threejs plugin - Top Menu > **Web > Qgis2threejs > Qgis2threejs Exporter**

![Ross-QGIS-Tutorial-59.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-59.png)

Select source heightmap and nothing else

RMB on heightmap > **Properties**

Set the **Resampling** level to **6**

Set **Resolution** to **400%**

![Ross-QGIS-Tutorial-60.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-60.png)

Enjoy browsing around your terrain in 3D:

![Ross-QGIS-Tutorial-61.png](https://www.rossedwards.co.uk/arma/tutorial/Ross-QGIS-Tutorial-61.png)

You can also export your terrain to a browser interface if you want - **File > Export to web**

**Coming Soon:**
  * Assisted image classification - for creation of mask
