# Geoprocesare în linie de comandă cu [GDAL/OGR](<http://gdal.org>)

### Seminariile [geo-spatial.org](http://www.geo-spatial.org/osgeo/timisoara2015)

### Timișoara, 22 noiembrie 2015

----------------------------

## 		I. Geoprocesare imagini

```sh
#~ Info despre un set de date
	#~ gdalinfo - report information about a file.
		gdalinfo srtm-L-34-048.tif
		gdalinfo -mm srtm-L-34-048.tif
			#~ -mm da valorile de min, max si (!) nodata

#~ Asociază sistemul de coordonate
	#~ gdal_translate - Copy a raster file, with control of output format.			
		gdal_translate -a_srs EPSG:31700 srtm-L-34-048.tif output/srtm-L-34-048-s42.tif
		gdalinfo output/srtm-L-34-048-s42.tif
		
		#~ aplică comanda în mod batch
		mkdir output
			
		for i in srtm-L-34-0??.tif; do gdal_translate -a_srs EPSG:31700 -a_nodata -32767 $i output/$i-s42.tif; done
			#~ (!) -a_nodata -32767
		cd output; rename .tif- "-" *.tif; ls; cd ..
			#~ am făcut puțină curățenie

```
