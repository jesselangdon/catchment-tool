# Catchment Tool
- Version: 1.0
- Revised 10-4-2016

## Summary
The **Catchment Tool** is an ArcGIS Python toolbox which currently includes one tool: *Delineate Catchments*. This tool was developed by [South Fork Research, Inc.](https://southforkresearch.org) to automate the process of delineating catchment area polygons for each stream reach or segment within a stream network. The catchment area polygons can then be used to calculate statistical values for geographically coincident spatial datasets representing environmental parameters.

Catchment areas represent all surfaces draining to a single point in a drainage basin (i.e. a pour point). Based on a set of pour points, the *Delineate Catchments* tool will delineate either upstream catchment area polygons (all areas upstream of a pour point within the analysis area, producing overlapping polygons), or reach catchment areas (all areas in the immediate vicinity, non-overlapping with adjacent drainage areas).  One of the key component in this process is the reconditioning of the digital elevation model (DEM) supplied by the user. The initial tool methods involved burning a stream network polyline feature class into a DEM, but later development switched to burning in a polygon representing open stream areas to recondition to better conform to the stream network. The stream area polygons can be derived from the USGS National Hydrography Dataset (NHD) [NHDArea](http://nhd.usgs.gov/userGuide/Robohelpfiles/NHD_User_Guide/Feature_Catalog/Hydrography_Dataset/NHDArea/NHD_Area.htm) stream features, or a bankfull ara polygon based on bankfull widths calculated for the stream network. It is recommended that the input stream network first be cleaned and edited so that is topologically correct, and split into user-defined segments of uniform length using the *Segment Stream Network* tool, which is part of the **Geomorphic Network and Analysis Toolbox** ([GNAT](http://bitbucket.org/KellyWhitehead/geomorphic-network-and-analysis-toolbox/wiki/Home)).

Although similar tools are available, such as ESRI's ArcHydro framework (Maidment, 2002) and the Multi-Watershed Delineation Tool (Chinnayakanahalli, K. et al., 2006), these tools do not allow overlapping, upstream catchment area delineation.

## Download
* [Catchment Tool version 1.0](https://github.com/jesselangdon/catchment-tool/archive/master.zip)
  * Download and unzip the file into a directory, then add the **Catchment Tool** toolbox into ArcGIS using `ArcToolbox > Add Toolbox`.

## Updates and Release Notes
11/9/2016
* Transferred repository from [jesselangdon](https://github.com/jesselangdon) to [SouthForkResearch](https://github.com/SouthForkResearch) 

10/05/2016
* Changed name from RCA Tools to Catchment Tool, uploaded v1.0
* The *Segment Stream Network* tool has been removed and incorporated into GNAT.

9/8/2016
* New version uploaded, v0.5
* Changes and revisions:
  * Added data pre-processing steps to the Segment Stream Network tool
  	* includes calculating stream order and branch IDs, using GNAT modules.
  * Added new module to more thoroughly clean up in_memory data sets.

8/8/2016
* New version uploaded, v0.4
* Changes and revisions include:
  * moved endpoint generation to Delineate Catchments tool. 
  * revised related inputs and outputs in tool interfaces.
  * improved alignment of catchment area polygons with stream segments.

8/3/2016
* New version uploaded, v0.3
* Changes and revisions include:
  * improved segmentation algorithm
  * added metadata output
  * minor interface improvements

7/28/2016
* Release RCA Tools, v0.1
  * Initial public release

## Requirements
* Catchment Tools
* ArcGIS 10.1, SP1
* Spatial Analyst
* Python 2.7.2
* GNAT (optional)

### ArcGIS Geoprocessing Requirements
* We recommend this tool be run with 64-bit python geoprocessing enabled.
* All inputs, including vector and raster data, are assumed to be in the same coordinate system and projection. UTM or North America Albers Equal Area projections are recommended.
* Remove M and Z values from the shape field of the input stream network feature class.

## Recommended Processing Workflow
### Data Preprocessing
1. Download the **Catchment Tool**, and unzip into a local directory
2. Add the **Catchment Tool** toolbox into ArcGIS using `ArcToolbox > Add Toolbox`
2. Project all input data (raster and vector) into the same projected coordinate system. Make sure the M and Z values are removed from the shape field.
3. Review stream network dataset (and edit if necessary):
    * connectivity of all stream reaches
    * correct flow direction
    * identify and remove all braids
    * (optional) segment the stream network, if desired, using the GNAT Segment Stream Network tool.

### Delineate Catchments Tool input/output
##### *Inputs*

* **Drainage area polygon**: Polygon feature class representing a single watershed or hydrologic unit (HUC) area encompassing the analysis area. *Required input*.

* **DEM**: Unprocessed DEM representing topography for the analysis area. This raster will be “reconditioned” using the stream network and (optionally) the bankfull polygon. *Required input*.

* **Stream network with a Branch ID**: Polyline feature class representing the stream network for the analysis area. This should be the stream network that is produced by the pre-processing workflow outlined above, which should include a Branch ID attribute field. *Required input*.

* **Segmented stream network**: Polyline feature class, output from the Segment Stream Network tool. *Required input*.

* **Stream area polygon**: Polygon feature class representing bankfull or open water stream areas. Can be used (in conjunction with the stream network dataset) to created a raster version of the stream network, which is then burned into the DEM as part of the reconditioning process. *Required input*.

* **Output file geodatabase**: The file geodatabase which will store the resulting catchment area polygons.

* **Upstream (i.e. overlapping) catchments?**: Indicates whether the entire drainage area upstream of each pour point will be delineated as the catchment,
 or if only the immediate reach catchment areas (non-overlapping) will be delineated.

##### *Outputs*

* **catch_final**: The delineated catchment area polygon feature class.
  * Fields Added:
    * OIDtmp: the original ObjectID value of the point record which served as a pour point for the catchment area.
    * sqkm: the calculated area value of the polygon, in square kilometers.
    * error: code indicating that an error occurred with the delineation process. 1 = improper placement of pour point

* **endpoints**: The point feature class representing stream segment endpoints. These are recommended for use as the pour points feature class input in the Delineate Catchments Tool.
  * Fields Added:
    * LineOID: the ObjectID of the stream segment from which the point was derived. This field can be used to join the endpoints feature class back to the segments feature class.

* **dem_recond**: Reconditioned DEM raster dataset. Provided for context.

### Known Issues
* **Sub-optimal pour point placement**: Unless the stream network that was used to generate pour points was directly derived from the DEM, there will inevitably be stream
segments (and their associated pour point) that will not spatially coincide precisely with areas of highest flow accumulation in the DEM.  This results in delineated catchments 
that are too small, and do not represent the full upstream drainage area.  Currently the Catchment Tool attempts to recondition the DEM by "burning in" the linear stream network and open stream area polygons
so that the DEM more closely conforms to the stream network.  Catchment area polygons with an area value below an arbitrarily chosen threshold
are tagged with an error code, to inform the user that these catchment areas may be suspect.

_Typical problem areas_:
  * Main stems / wide flood plains
  * Areas with dense stream networks
  
* **Long run time**: Selecting the upstream catchment option results in drastically longer processing times.

## Citations
* Chinnayakanahalli, K., C. Kroeber, R. Hill, J. Olson, D. G. Tarboton and C. Hawkins, (2006). Manual for Regional Watershed Analysis Using the Multi-Watershed Delineation Tool. Utah State University. 

* Maidment, D. R. (2002). Arc Hydro: GIS for water resources (Vol. 1). ESRI, Inc.

## Acknowledgements
The Catchment Tool toolbox is under development for [South Fork Research, Inc.](http://southforkresearch.org) by Jesse Langdon.
