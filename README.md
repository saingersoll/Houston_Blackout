# Houston TX Blackout (2021)

"In February 2021, the state of Texas suffered a major power crisis, which came about as a result of three severe winter storms sweeping across the United States on February 10--11, 13--17, and 15--20."[^1] For more background, check out these [engineering](https://www.youtube.com/watch?v=08mwXICY4JM&ab_channel=PracticalEngineering) and [political](https://www.youtube.com/watch?v=Zcrsgdl_hP0&ab_channel=Vox) perspectives.

[^1]: Wikipedia. 2021. "2021 Texas power crisis." Last modified October 2, 2021. <https://en.wikipedia.org/wiki/2021_Texas_power_crisis>.

For this project, we intend to:
- estimate the number of homes in Houston that lost power as a result of the first two storms
- investigate if socioeconomic factors are predictors of communities recovery from a power outage

Using visual aids:
-  Map of median income by census tract, designating which tracts had blackouts.
-  Map of median income by census tract, designating which tracts had blackouts with bubbles.
-  Histogram of the distribution of income in impacted and unimpacted tracts.
-  Boxplot of the distribution of income in impacted and unimpacted tracts.

##### Learning objectives:

-   load vector/raster data
-   simple raster operations
-   simple vector operations
-   spatial joins

### About the Data
#### Night Lights Data

The analysis is based on remotely-sensed night lights data, acquired from the [Visible Infrared Imaging Radiometer Suite (VIIRS)](https://en.wikipedia.org/wiki/Visible_Infrared_Imaging_Radiometer_Suite) onboard the Suomi satellite. In particular, you will use the VNP46A1 to detect differences in night lights before and after the storm to identify areas that lost electric power.

Use NASA's Worldview to explore the data around the day of the storm. There are several days with too much cloud cover to be useful, but 2021-02-07 and 2021-02-16 provide two clear, contrasting images to visualize the extent of the power outage in Texas.

VIIRS data is distributed through NASA's [Level-1 and Atmospheric Archive & Distribution System Distributed Active Archive Center (LAADS DAAC)](https://ladsweb.modaps.eosdis.nasa.gov/). Many NASA Earth data products are distributed in 10x10 degree tiles in sinusoidal equal-area projection. Tiles are identified by their horizontal and vertical position in the grid. Houston lies on the border of tiles h08v05 and h08v06. We therefore need to download two tiles per date.

 To prevent this project from being a large data wrangling challenge, this data has been downloaded and prepped as the following files for you to work with, stored in the `VNP46A1` folder.

-   `VNP46A1.A2021038.h08v05.001.2021039064328.h5.tif`: tile h08v05, collected on 2021-02-07
-   `VNP46A1.A2021038.h08v06.001.2021039064329.h5.tif`: tile h08v06, collected on 2021-02-07
-   `VNP46A1.A2021047.h08v05.001.2021048091106.h5.tif`: tile h08v05, collected on 2021-02-16
-   `VNP46A1.A2021047.h08v06.001.2021048091105.h5.tif`: tile h08v06, collected on 2021-02-16

#### Roads Data

Typically highways account for a large portion of the night lights observable from space (see Google's [Earth at Night](https://earth.google.com/web/@27.44405464,-84.7693044,206.63660162a,8916361.52264659d,35y,0h,0t,0r/data=CiQSIhIgMGY3ZTJkYzdlOGExMTFlNjk5MGQ2ZjgxOGQ2OWE2ZTc)). To minimize falsely identifying areas with reduced traffic as areas without power, we will ignore areas near highways.

[OpenStreetMap (OSM)](https://planet.openstreetmap.org/) is a collaborative project which creates publicly available geographic data of the world. Ingesting this data into a database where it can be subsetted and processed is a large undertaking. Fortunately, third party companies redistribute OSM data. We used [Geofabrik's download sites](https://download.geofabrik.de/) to retrieve a shapefile of all highways in Texas and prepared a Geopackage (`.gpkg` file) containing just the subset of roads that intersect the Houston metropolitan area. 

-   `gis_osm_roads_free_1.gpkg`

#### Buildings Data

We can also obtain building data from OpenStreetMap. We downloaded from Geofabrick and prepared a GeoPackage containing only houses in the Houston metropolitan area.

-   `gis_osm_buildings_a_free_1.gpkg`

#### Socioeconomic Data

We cannot readily get socioeconomic information for every home, so instead we obtained data from the [U.S. Census Bureau's American Community Survey](https://www.census.gov/programs-surveys/acs) for census tracts in 2019. The *folder* `ACS_2019_5YR_TRACT_48.gdb` is an ArcGIS ["file geodatabase"](https://desktop.arcgis.com/en/arcmap/latest/manage-data/administer-file-gdbs/file-geodatabases.htm), a multi-file proprietary format that's roughly analogous to a GeoPackage file.

The geodatabase contains a layer holding the geometry information, separate from the layers holding the ACS attributes. We will combine the geometry with the attributes to get a feature layer that `sf` can use. To determine the number of homes that lost power, we will link (spatially join) these areas with [OpenStreetMap](https://www.openstreetmap.org/#map=4/38.01/-95.84) data on buildings and roads.

**Note:** the data associated with this project is too large to include in the GitHub repo. Instead, download data from [here](https://drive.google.com/file/d/1bTk62xwOzBqWmmT791SbYbHxnCdjmBtw/view?usp=sharing). Unzip the folder and all the contents and store in your directory as follows. Don't include data when you push to GitHub!

    Houston_Blackout
    │   README.md
    │   Rmd/Proj files    
    │
    └───data
        │   gis_osm_buildings_a_free_1.gpkg
        │   gis_osm_roads_free_1.gpkg
        │
        └───ACS_2019_5YR_TRACT_48_TEXAS.gdb
        |   │   census tract gdb files
        |
        └───VNP46A1
        |   │   VIIRS data files
