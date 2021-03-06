# GeoServer Workflow
This script automates the process for preparing GeoTIFFs for use in WMS applications and adding them to GeoServer.

This script is specific to Emory University's workflow and infrastructure, but could be adapted. Please feel free to open an issue here or contact <libsysdev-l@listserv.cc.emory.edu> if you have questions.

Special thanks to Eric Willoughby at Georgia State University for working out the GDAL commands and consulting on GeoServer REST calls.

## Dependencies
Install the [GDAL](http://www.gdal.org/) tools for your OS: [https://trac.osgeo.org/gdal/wiki/DownloadingGdalBinaries](https://trac.osgeo.org/gdal/wiki/DownloadingGdalBinaries)
You will need ruby and a few gems:
> If you are using OS X you will need to install [XCode](https://developer.apple.com/xcode/) and the [XCode Command Line Tools](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/) before trying to install the gems.

<code>gem install net-sftp</code>

<code>gem install httparty</code>

<code>gem install nokogiri</code>

<code>gem install highline</code>

## config.yaml
Provided is a sample config file (<code>config.yaml.dst</code>). Rename that to <code>config.yaml</code> and fill in the various items.

## What the script does
* Collects all the tiff files in the configured directory
* Runs each tiff though the following process one at a time
	* Uses GDAL commands to prepare for WMS applications
	* Finds the proper metadata file based on configured directory and matching file name. For example, if the script finds Sheet4.tif, it will look for Sheet4.xml
	* Checks the metadata file for an ARK. If no ARK is found, it creates one and adds it to the metadata file
	* Uploads the GeoTIFF to the configured remote directory
	* Adds a store in GeoServer for the GeoTIFF using the ARK as the `Name` and the title from the metadata file for the `Title`
	* Adds a layer in GeoServer for the new store
	* Edits/updates the layer's fields in GeoServer based on the metadata

## Determining the EPSG Code of Your Geospatial Data
Geoserver uses EPSG codes to classify coordinate reference systems (geographic or projected). To use this script, you will need to know the current EPSG code for your data’s CRS, as well as the desired one if you’d like to change it. In ArcGIS, the EPSG code can usually be found in the Spatial Reference Properties box (highlighted below):

![](https://s3.amazonaws.com/atlmaps-prod/properties.png)

In QGIS, the EPSG code can usually be found in the Layer Properties box (highlighted below):
![](https://s3.amazonaws.com/atlmaps-prod/properties2.png)If you are having difficulty determining an EPSG code, the makers of Geoserver have also created a handy tool to help: http://prj2epsg.org/search

## Usage
If you do not pass any options to the script, it will run though the process described above. You will be prompted for input and output coordinate systemes (EPSG codes). It will default to the ones you set in the `config.yaml`. To use the configured codes, simply tap enter when prompted. Otherwise, you can override your configured code. See section above about EPSG codes. You can also run the sript with the `-y` flag to accept the defaults without being prompted (good for automating or running a large batch at once).

You can pass the a specific path to a GeoTIFF with the `-t` flag. You can also pass it a specific path for a metadata file with the `-m` flag. If you do not provide a metadata file, the script will try to find it based on the file's name. **Note**: You can only specify a metadata file when specifying a GeoTIFF.

You can also run a single part of the script with the `-m [name of metod]`. Below are all the optins and names of each method:

	ruby tada [options]
		-t, --tif /path/to/map.tif         Path to tif file.
    	-d, --mdfile /path/to/metadata.xml Path to metadata file.
      	-m, --method METHOD                Run a single method.
		-y, --default_cs                   Use the coordinate systems form config file.
       Methods:
       	process # Runs the GeoTIFF thorugh GDAL process
       	upload # Uploads GeoTIFF to GeoServer
       	add_store # Adds a store in GeoServer for the GeoTIFF
       	add_layer # Creates a layer from GeoTIFF's store in GeoServer
       	update_layer # Updates the layer's metadata in GeoServer

## License
This GeoServer-Workflow is distributed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0). Enjoy.
