Slopes experiments (WIP)
================

This repo structures and documents the activities in the creation of a map with the slopes of a road network, replicating the rationale, code and most of the instructions in: (https://github.com/U-Shift/Declives-RedeViaria/blob/main/README.md) and adding other relevant information to me to understand the process, successfully create slopes maps and achieve the desired goals.

**Goals**
- check if the road network of a given city is smooth for cycling in terms of slope values
- explore QGIS functionalities
- start learning R language

**References**
- https://github.com/temospena/slopes
- https://docs.ropensci.org/slopes/articles/slopes.html#calculate-slope
- https://github.com/U-Shift/Declives-RedeViaria/blob/main/README.md
- http://www.rosafelix.bike/
- https://r.geocompx.org/spatial-class.html
- https://web.tecnico.ulisboa.pt/rosamfelix/r/COMPILACAO.html#1_scripts_b%C3%A1sicos

**Input map files**
- Digital Elevation Model (DEM) of the country (raster)
- administrative divisions of the country (vector)
- road network of the country (vector)

**Output map files**
- map of slope classes of the road network of a municipality (*html* file):

## Create map of slope classes of a municipality road network

**Process overview**

![plot](./images/diagram_process_overview.png)

In this example:
- country: Portugal
- mun: Ovar

### Software requirements
*QGIS*
- Install it by following the instuctions in https://qgis.org/resources/installation-guide/.

*R project*
1. Install it by following the instructions in https://cran.r-project.org/.
   For Windows, download the latest version of R from https://cran.r-project.org/bin/windows/base/.
3. Install the required packages by running the following command in R:
    ```
    install.packages("<package_name>")
    ```

    | Package name | Brief description | Source of more information |
    | ------------ | ----------------- | ---------------------------|
    | dplyr  | A fast, consistent tool for working with data frame like objects, both in memory and out of memory. | https://cran.r-project.org/web/packages/dplyr/index.html |
    | geodist | Fast, dependency-free geodesic distance calculations | https://cran.r-project.org/web/packages/geodist/index.html |
    | raster | Reading, writing, manipulating, analyzing and modeling of spatial data; superseded by the "terra" package | https://cran.r-project.org/web/packages/raster/index.html |
    | sf | Simple Features for R; a standardized way to encode and analyze spatial vector data | https://cran.r-project.org/web/packages/sf/index.html |
    | slopes | Calculates the slope (longitudinal gradient or steepness) of linear geographic features such as roads and rivers | https://cran.r-project.org/web/packages/slopes/index.html |
    | spData | Diverse spatial datasets for demonstrating, benchmarking and teaching spatial data analysis | https://cran.r-project.org/web/packages/spData/index.html ? |
    | terra | Methods for spatial data analysis with vector and raster data | https://cran.r-project.org/web/packages/terra/index.html ? |
    | tmap | Thematic maps, that is, geographical maps in which spatial data distributions are visualized | https://cran.r-project.org/web/packages/tmap/index.html |

### 1. Download map with municipality limits of the country
Download the most recent CAOP (Carta Administrativa Oficial de Portugal) *`CAOP_Continente_2024_1-gpkg.zip`* by selecting the https://geo2.dgterritorio.gov.pt/caop/CAOP_Continente_2024_1-gpkg.zip link.

### 2. Create map of the selected municipality
1. Unzip the downloaded *geopackage* file *`CAOP_Continente_2024-gpkg.zip`*.
2. Open R program and load the CAOP file *`Continente_CAOP2024.gpkg`*:
   ```
   library(sf)
   CAOP = st_read("<folder_path>/Continente_CAOP2024.gpkg", layer = 'cont_municipios')
   ```
   ```
   ## Reading layer `cont_municipios' from data source `<folder_path>\Continente_CAOP2024.gpkg' using driver `GPKG'
   ## Simple feature collection with 278 features and 9 fields
   ## Geometry type: MULTIPOLYGON
   ## Dimension:     XY
   ## Bounding box:  xmin: -119191.4 ymin: -300404.8 xmax: 162129.1 ymax: 276083.8
   ## Projected CRS: ETRS89 / Portugal TM06
   ```
      
3. Check the names of the columns, particularly **municipalities** and **geometry** ones, to be used in the next step:
   ```
   colnames(CAOP)
   ```
   ```
   ## [1] "dtmn"          "municipio"     "distrito_ilha" "nuts3"         "nuts2"         "nuts1"         "area_ha"       "perimetro_km"  "n_freguesias"  "geom"
   ```
   
4. Create geometry with the desired columns:
   ```
   municips_PT = CAOP[,c("municipio","geom")]
   ```
  
5. Change CRS from ETRS89 / Portugal TM06 to WGS84:
   ```
   municips_PT = st_transform(municips_PT, 4326)
   ```

6. Save geometry as a *geopackage* file:
    ```
    st_write(municips_PT, "<path>/municips_PT.gpkg", append=F)
    ```
    ```
    ## Writing layer `MunicipsPT' to data source `<folder_path>/MunicipsPT.gpkg' using driver `GPKG'
    ## Writing 278 features with 1 fields and geometry type Multi Polygon.
    ```
    To reproduce the exercise for another municipality, skip the steps 1 to 6 and use the file *`municips_PT.gpkg`* for the next steps.

7. Get the list of the municipalities:
    ```
    municips_PT$municipio
    ```

8. Pick the name of the desired municipality - "Ovar" in this example - and create map *`Ovar_limit.gpkg`*:
    ```
    library(dplyr)
    Ovar_limit = municips_PT %>% filter(municipio == "Ovar")
    st_write(Ovar_limit, "D:/Documentos/Projetos/GIS/final_github_slopes-experiments/Ovar_limit.gpkg")
    ```
    

### 3. Get road network of the country
1. In R, get road network from OSM:
    ```
    library(osmextract)
    library(sf)
    portugal_osm = oe_get("Portugal", provider = "geofabrik", stringsAsFactors = FALSE, quiet = FALSE,
                       force_download = TRUE, force_vectortranslate = TRUE)
    ```
    ```
    ## The input place was matched with: Portugal
    ## Downloading the OSM extract                                                                               File downloaded!
    ## Starting with the vectortranslate operations on the input file!
    ## 0...10...20...30...40...50...60...70...80...90...100 - done.
    ## Finished the vectortranslate operations on the input file!
    ## Reading layer `lines' from data source `/<temp_path>/geofabrik_portugal-latest.gpkg' using driver `GPKG'
    ## Simple feature collection with 1984239 features and 10 fields
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: -35.65957 ymin: 28.11586 xmax: -2 ymax: 51.28145
    ## Geodetic CRS:  WGS 84
    ```
    The road network of Portugal is downloaded to a temporary folder in *geopackage* format, native format from QIGS, equivalent to shapefile format.

2. Read *geopackage* file and check data, such as Coordinate Reference System (CRS) and geometry type:
    ```
    networkOSM_PT = st_read("<folder_path>/geofabrik_portugal-latest.gpkg", layer= "lines")
    ```
    ```
    ## Reading layer `lines' from data source `<folder_path>\geofabrik_portugal-latest.gpkg' using driver `GPKG'
    ## Simple feature collection with 1984239 features and 10 fields
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: -35.65957 ymin: 28.11586 xmax: -2 ymax: 51.28145
    ## Geodetic CRS:  WGS 84

3. Check available categories for roads, that is, the existing values for key **highway=***, and respective definitions in https://wiki.openstreetmap.org/wiki/Key:highway):
   ```
   table(networkOSM_PT$highway)
   ```
   
4. Filter roads with the desired categories:
    ```
    library(dplyr)
    networkOSM_PT_filtered = networkOSM_PT %>% 
    dplyr::filter(highway %in% c('primary', "primary_link", 'secondary',"secondary_link", 'tertiary', "tertiary_link",
                "trunk", "trunk_link", "residential", "cycleway", "living_street", "unclassified",
                "motorway", "motorway_link", "pedestrian", "steps", "service", "track"))
    ```
    *Notes from reference instructions:* OpenStreetMap classifies the roads in different categories. The footpaths should be left out of the selected network sample. Also, to get a lighter network, only the roads
    with higher levels can be selected, such as the ones with categories **primary**, **secondary** and **tertiary**. Some roads do not have a category assigned yet ( value is **unclassified**) but can be used for cycling as well. I kept this category.

6. Save filtered network of the country as a geopackage file - *`networkOSM_PT_filtered.gpkg`*:
    ```
    st_write(networkOSM_PT_filtered, "<path>/networkOSM_PT_filtered.gpkg")
    ```

### 4. Clip road network by the municipality
1. In R, crop the road network to make the next operation lighter, using the municipality limit:
    ```
    #municips_PT = st_read("<path>/MunicipsPT.gpkg")
    library(stplanr)
    linesOSM_Ovar = st_crop(networkOSM_PT_filtered, Ovar_limit)
    ```
    ```
    ## Warning message:
    ## attribute variables are assumed to be spatially constant throughout all geometries
    ```

2. Clip the road network by using a buffer of 100 m, for example, to avoid cutting the lines that are in the limit of the municipality:
    ```
    networkOSM_Ovar = st_intersection(linesOSM_Ovar, geo_buffer(Ovar_limit, dist=100))
    ```
    ```
    ## Warning message:
    ## attribute variables are assumed to be spatially constant throughout all geometries
    ```

3. Save the resulting geometry as *geopackage* file - *`networkOSM_Ovar.gpkg`*:
    ```
    st_write(networkOSM_Ovar, "<folder_path>/networkOSM_Ovar.gpkg")
    ```

### 5. Delete unconnected segments
The segments of the geometry that are isolated, that is, that not connected to the main road network, must be deleted:
1. Load the layer *`networkOSM_Ovar.gpkg`* in QGIS.
2. In the upper menu, select **Vector** and check if **Disconnected Islands** plugin is displayed.
   - If it is, go to step 3.
   - If not, in the upper menu, select **Plugins** > **Manage and Install Plugins** and check the **Installed** plugins.
       - If **Disconnected Islands** is on the list, select its box for it to appear on the **Vector** plugins list and go to step 3. (CHECK)
       - If **Disconnected Islands** is not on the list, install it.
           1. Select **Not installed**.
           2. Enter the plugin name name in the search box.
           3. Select it and then select **Install Plugin**.
3. Hover over **Disconnected Islands** and then select **Check for Disconnected Islands**.
4. Select the option **Use all vertices on a road link** and the lowest tolerance in **Tolerance**. Select **OK**.
   Output: 203 segments were selected, with a group ID assigned higher than 0 (networkGRP attribute).
6. Select all the segments with a networkGRP > 0:
   1. Right-click the **networkOSM_Ovar** layer and select **Open Attribute Table**.
   2. Select the **Select features using an expression** icon.
   3. On the **Expression**  tab, enter the text "networkGrp > 0" and select **Select Features**.
7. On the **Attribute table** window, select the **Invert selection** icon.
8. Export selection as a new geopackage file:
   1. Right-click the **networkOSM_Ovar** layer, hover over **Export** and select **Export Selected Features As**.
   2. On the **Save Vector Layer as** window, enter the desired folder and name of file *`networkOSM_Ovar_cleaned.gpkg`* in the **Filename**, check CRS and select **OK**.

### 6. Convert network geometry to the required type
1. Open R and load the cleaned road network from Ovar:
    ```
    networkOSM_Ovar_cleaned = st_read("<path>/networkOSM_Ovar_cleaned.gpkg")
    ```
    ```
    ## Reading layer `networkosm_ovar' from data source 
    ## `<folder_path>\networkOSM_Ovar_cleaned.gpkg' 
    ##  using driver `GPKG'
    ## Simple feature collection with 6838 features and 12 fields
    ## Geometry type: MULTILINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: -8.691657 ymin: 40.81252 xmax: -8.523237 ymax: 40.97621
    ## Geodetic CRS:  WGS 84
    ```
    After the operation from step 5, geometry type is now MULTILINESTRING. However for slopes calculation, the type must be LINESTRING.
   
3. Create a new layer with the same features of **newtowrkOSM_Ovar_cleaned** but with correct geometry type LINSTRING, using the geometries before the cleansing of step 5:
    ```
    networkOSM_Ovar_conv = networkOSM_PT_filtered %>% filter(osm_id %in% networkOSM_Ovar_cleaned$osm_id)
    ```
4. Check if geometry type is now correct:
    ```
    st_geometry(networkOSM_Ovar_conv)
    ```
    ```
    ## Geometry set for 6838 features 
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: -8.691792 ymin: 40.78936 xmax: -8.509474 ymax: 40.98472
    ## Geodetic CRS:  WGS 84
    ## First 5 geometries:
    ## LINESTRING (-8.610939 40.92746, -8.610506 40.92...
    ## LINESTRING (-8.602149 40.93018, -8.602284 40.92...
    ## LINESTRING (-8.60018 40.9277, -8.60056 40.92781...
    ## LINESTRING (-8.603908 40.92804, -8.6037 40.9279...
    ## LINESTRING (-8.60317 40.92573, -8.602993 40.925...
    ```
5. Save the new layer as *geopackage* file *`networkOSM_Ovar_conv.gpkg`*:
    ```
    st_write(networkOSM_Ovar_conv, "<folder_path>/networkOSM_Ovar_conv.gpkg")
    ```
    
### 7. Cut long segments
The goal is to cut long segments to calculate a mean value that is more realistic. Road segments will be cut at the instersection with another segments that have the same z level, to avoid cutting brunels. EXPLAIN
1. In R, check the number of rows of the road network layer:
    ```
    nrow(networkOSM_Ovar_conv)
    ```
    ```
    ## [1] 6838
    ```
2. Use a function that cut the segments in its internal vertices except in the intersection with brunels (bridges and tunnels):
    ```
    library(stplanr)
    network_Ovar = stplanr::rnet_breakup_vertices(networkOSM_Ovar_conv)
    ```
3. Check the number of rows after cutting the road network in the intersections:
    ```
    nrow(network_Ovar)
    ```
    ```
    ## [1] 11772
    ```
4. Export the resulting geometry to *geopackage* format *`network_Ovar.gpkg`*:
   
    ```
    st_write(network_Ovar, "<folder_path>/network_Ovar.gpkg")
    ```
    The road network is ready for the slopes calculation.

### 8. Download DEM of the country
Download the QGIS project with DEM layer from https://www.fc.up.pt/pessoas/jagoncal/dems/ by selecting the https://www.fc.up.pt/pessoas/jagoncal/dems/dems_pt.zip link.

### 9. Clip DEM by the road network
Since the raster covers the country but only a small area is needed, cut the DEM raster to the city netowrk for the slopes analysis.

1. After unzipping the downloaded file, double-click in *`dems_pt.qgz`* to open it in QGIS.
2. Choose the desired DEM (I selected STRM).
3. In the upper menu, select **Raster** > **Extraction** > **Clip Raster by Extent**.
4. In the **Raster Extraction - Clip Raster by Extent** window, select the parameters:
    - **Input layer**: select DEM file from the dropdown list
    - **Clipping extent**: select **Draw on Map Canvas** from the dropdown list and draw the rectangle of the desired extent
5. Select **Run**. The clipped raster layer is created.
6. Select the clipped DEM layer and right-click it.
7. Hove over **Export** and select **Save as** from the menu.
8. In the **Save raster layer as** window, select **GeoTIFF** from the **Format** drop-down list.
9. In the **Filename** box, enter the name of the file (I chose the name *DEM_Ovar*.) You can select the side button **Navigate** to choose the folder and alterantively enter the name in the pop-up window.
10. Select **OK**. File with name *`DEM_Ovar.tif`* is created. \
*NOTE:* The default CRS WGS84 can be selected in the **CRS** field so that the DEM raster is in the same Coordinate Reference System as the one of the road network. This will be important in section [6. Calculate slopes](#calculate-slopes).

### 10. Check geometry requirements and visualize
The DEM and road network geometries - *`DEM_Ovar.tif`* and *`network_Ovar.gpkg`* - must be in the same CRS:
1. Load the clipped DEM in R:
    ```
    library(raster)
    DEM_Ovar = raster("<folder_path>/DEM_Ovar.tif")
    ```

2. Check some data of raster:
   ```
   class(DEM_Ovar)
   ```
   ```
   ## [1] "RasterLayer"
   ## attr(,"package")
   ## [1] "raster"
   ```
   ```
   summary(values(DEM_Ovar))
   ```
   ```
   ## Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
   ##-7.00    0.00   26.00   69.32  124.00  353.00    9357
   ```
   ```
   res(DEM_Ovar)
   ```
   ```    
   ## [1] 0.0002950317 0.0002264756
   ```
   
4. Plot DEM and road network of Ovar together:
    ```
    raster::plot(DEM_Ovar)
    plot(sf::st_geometry(network_Ovar), add = TRUE)
    ```

![plot](./images/DEM_and_network_Ovar.png)

### 11. Calculate slopes and statistics of the road network
1. Still in R, add a column to the road network with the slopes values:
    ```
    library(slopes)
    network_Ovar$slope = slope_raster(network_Ovar, dem = DEM_Ovar )
    ```
   
3. Calculate the percentages of slope values and key summary statistics - minimum, P25, median, average, P75, maximum:
    ```
    network_Ovar$slope_perc = network_Ovar$slope*100
    summary(network_Ovar$slope_perc)
    ```
    ```
    ## Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ## 0.000   1.279   2.469   3.175   4.165  38.185
    ``` 

### 12. Assign slope classes to the network and calculate percentages
1. Create a new column for slope classes, here called gradients, and assign a value for each road:
    ```
    network_Ovar$gradient =  network_Ovar$slope_perc %>%
       cut(
         breaks = c(0, 3, 5, 8, 10, 20, Inf),
         labels = c("0-3: flat", "3-5: mild","5-8: medium", "8-10: hard", "10-20: extreme", ">20: impossible"),
         right = F
       )
    ```

2. Calculate the percentage of roads of each slope class:
    ```
    round(prop.table(table(network_Ovar$slope_class))*100,1)
    ```
    ```
    ##       0-3: flat       3-5: mild     5-8: medium      8-10: hard  10-20: extreme >20: impossible 
    ##            59.0            22.6            12.2             2.9             3.2             0.1 
    ``` 
    This means that more than half of the streets of the sample are flat or almost flat, and more than 80% of the streets are perfectly cyclable.

### 13. Calculate length of the network and check values
1. Add a column with the length of each segment of the geometry in meters:
    ```
    network_Ovar$length = st_length(network_Ovar)
    ```
2. Check the new columns and some values:
    ```
    head(network_Ovar)
    ```

### 14. Create interactive map of network with the slope classes
1. Prepare data for visualization by creating a colour palette between dark green and dark red:
    ```
    palredgreen = c("#267300", "#70A800", "#FFAA00", "#E60000", "#A80000", "#730000")
    ```
2. Load library and set tmap mode to "view":
    ```
    library(tmap)
    tmap_mode("view")
    ```
    ```
    ## tmap mode set to "view".
    ```
3. Create map of slope classes:
    ```
    tmap_options(basemaps = leaflet::providers$CartoDB.Positron)
    mapslopes =
     tm_shape(network_Ovar) +
     tm_lines(
         col = "gradient",
         palette = palredgreen, #colours palette
         lwd = 2, #thickness of lines
         title.col = "Gradient [%]",
         popup.vars = c("Type: " = "highway",
                        "Length" = "length",
                        "Slope: " = "slope_perc",
                        "Slope class: " = "gradient"),
         popup.format = list(digits = 1),
         # id = "slope"
         id = "name" #if the PC is not able to export due lack of memory, delete this line
       )
    mapslopes
    ```

4. Save created map as an *html* file *`slopes_SRTM_Ovar.html`*:
    ```
    tmap_save(map_slopes, "<folder_path>/slopes_SRTM_Ovar.html")
    ```

    ![plot](./images/html_map_slopes_Ovar.png)
