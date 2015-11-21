# Geoprocesare în linie de comandă cu [GDAL/OGR](<http://gdal.org>)

### Seminariile [geo-spatial.org](http://www.geo-spatial.org/osgeo/timisoara2015)

### Timișoara, 22 noiembrie 2015

----------------------------

## 		I. Geoprocesare imagini

```sh
#~ /bin/.sh

#~ http://gdal.org/gdal_utilities.html
#~ http://gdal.org/gdal_datamodel.html

#~ +++++++++++++++++++++++++++++++
#~ Partea I. Geoprocesare imagini

#~ 1.Info despre un set de date
	#~ gdalinfo - report information about a file.
		gdalinfo tm_srtm-L-34-079.tif
		#~ (!)LOCAL_CS
		#~ Pixel Size
		#~ IMAGE STRUCTURE METADATA
		#~ Corner Coordinates -- par Stereo70
		gdalinfo -mm tm_srtm-L-34-079.tif
			#~ -mm da valorile de min, max
			#~ Computed Min = -32767.000

#~ 2.Asociază sistemul de coordonate
	#~ gdal_translate - Copy a raster file, with control of output format.	
		mkdir output		
		
		gdal_translate -a_srs EPSG:3844 -a_nodata -32767 tm_srtm-L-34-079.tif output/tm_srtm-L-34-079-s42.tif
			#~ -a_srs: 3844 = cod EPSG pt Stereo70, vezi http://www.spatialreference.org/ref/epsg/3844/
			#~ (!) -a_nodata -32767
		gdalinfo -mm output/tm_srtm-L-34-079-s42.tif
			#~ LOCAL_CS["Pulkovo 1942(58) / Stereo70",
			#~ NoData Value=-32767
			#~ Computed Min/Max=74.000,260.000
		
		#~ aplică comanda în mod batch
		
		for i in srtm-L-34-0??.tif; do gdal_translate -a_srs EPSG:3844 -a_nodata -32767 $i output/$i-s42.tif; done
			#~ (!) -a_nodata -32767
		cd output; rename .tif- "-" *.tif; ll; cd ..
			#~ am făcut puțină curățenie
		
#~ 3.Transformă din .tif in .img
		gdal_translate --formats
			#~ denumirea prescurtata a formatelor ex. HFA pt Erdas Imagine .img
		gdal_translate --formats | grep -i .web # sau TIFF, sau ASC
			#~ verifică dacă un anumit format de date este suportat
		gdal_translate --format HFA # sau GTiff, sau AAIGrid
		gdal_translate --format webp
			#~ află detalii despre un anumit format
			#~ (!) opțiunile din <CreationOptionList> vor merge in -co, vezi mai jos 
		
		gdalinfo -mm tm_landsat321-L-34-079.tif
			#~ din Image Structure Metadata lipsește COMPRESSION -> nu are compresie
			#~ detalii la http://trac.osgeo.org/gdal/wiki/rfc14_imagestructure
			#~ (!) no_data=0 pe fiecare bandă
		gdal_translate
		
		gdal_translate -of hfa -b 3 -b 2 -b 1 -co compressed=yes tm_landsat321-L-34-079.tif output/tm_landsat321-L-34-079.img
			#~ -co = creation option
			#~ (!) -co compressed=yes
		gdalinfo -mm output/tm_landsat321-L-34-079.img
		#~ COMPRESSION=RLE detalii la https://en.wikipedia.org/wiki/Run-length_encoding
		#~ (?) LAYER_TYPE=athematic
		
		gdal_translate -of webp -b 3 -b 2 -b 1 tm_landsat321-L-34-079.tif output/tm_landsat321-L-34-079.webp
		gdalinfo -mm output/tm_landsat321-L-34-079.webp
		
		#~ în mod batch
		for i in landsat321-L-34-0??.tif; do gdal_translate -of webp -b 3 -b 2 -b 1 -a_srs EPSG:3844 $i output/$i.webp; done
			
		cd output; rm *.xml; rename .tif. "." *.img; ll; cd ..
			#~ puțină curățenie
		
		gdalinfo -mm output/tm_landsat321-L-34-079.img
		
		
#~ ex. 1 Transformati seturile de date srtm in Arc/Info ASCII GRID.
#~ (10 min)

		
#~ 4.Combină mai multe benzi într-un singur rastru
	#~ gdalbuildvrt - Build a VRT from a list of datasets.
		gdalinfo -mm B1-L-34-079.tif
			 #~ Pixel Size = (!)rezolutia.
		gdalinfo -mm B61-L-34-079.tif
		gdalinfo -mm BPan-L-34-079.tif
			#~ detalii la http://landsat.usgs.gov/band_designations_landsat_satellites.php
		
		gdalbuildvrt
		gdalbuildvrt -separate -vrtnodata 0 -overwrite landsat123457-L-34-048.vrt B?-L-34-079.tif
			#~ ? wildcard
			#~ (!) VRT: http://www.gdal.org/gdal_vrttut.html
			#~ parametru cheie -separate
			#~ am combinat doar benzile cu aceeasi rezolutie, fără 61 și Pan
		gdalinfo -mm landsat123457-L-34-048.vrt
			#~ structura VRT, fișiere input trebuie să existe în același folder
	
		gdal_translate --format GTiff
			#~ trecere în revistă pt -co; detalii la http://www.gdal.org/frmt_gtiff.html
		gdal_translate -co compress=deflate -a_nodata 0 landsat123457-L-34-048.vrt output/landsat123457-L-34-048.tif
			
		gdalinfo -mm output/landsat123457-L-34-048.tif
			#~ numărul de benzi
			#~ puțină joacă în QGIS cu combinații de benzi
			#~ http://web.pdx.edu/~emch/ip1/bandcombinations.html
			
	
#~ 5.Reproiecteaza fiecare rastru in: 
#~ 		- Lat-Long/WGS84: EPSG:4326
#~ 		- UTM/WGS84, zona 34 N: EPSG:32634
#~ 		- Gauss-Krueger/Pulkovo1942, zona 4: EPSG:28404
	
	#~ gdalwarp - Warp an image into a new coordinate system.
		
		gdalwarp
		
		gdalwarp -s_srs EPSG:3844 -t_srs EPSG:28404 -dstnodata -9999 -r cubic -wm 1024 -multi -co compress=deflate output/srtm-L-34-034-s42.tif output/tm_srtm-L-34-079-gk.tif
			#~ parametri cheie -s_srs si -t_srs
			#~ (!)EPSG
			#~ despre datumuri si transformari
			#~ http://spatialreference.org/ref/epsg/3844/
			#~ http://www.ancpi.ro/pages/wiki.php?lang=ro&pnu=transformariCoordonate
			#~ http://www.globalmapper.com/helpv12/datum_list.htm
		gdalinfo -mm output/tm_srtm-L-34-079-gk.tif
			#~ COMPRESSION=DEFLATE (!) lossless pentru dem-uri
		
		for i in output/srtm-L-34-0??-s42.tif; do gdalwarp -s_srs EPSG:3844 -t_srs EPSG:28404 -dstnodata -9999 -r cubic -wm 1024 -multi -co compress=deflate $i $i-gk.tif; done
			
		cd output; rename s42.tif-gk "gk" *-gk.tif; ll; cd ..
			#~ curățenie
	
#~ 6. Combină raștrii într-un mozaic optimizat
	
	#~ gdalbuildvrt - Build a VRT from a list of datasets.
	ls landsat321-*.tif > file.list
		#~ lista seturilor de date care vor contribui la mozaic
	
	gdalbuildvrt
	gdalbuildvrt -srcnodata "0 0 0" -vrtnodata "-9999 -9999 -9999" -input_file_list file.list landsat321Apuseni.vrt
		#~ -src_nodata si -vrt_nodata: valoarea pe fiecare banda
	gdalinfo -mm landsat321Apuseni.vrt
	
	#~ compresie (!)deflate
	gdal_translate -a_srs EPSG:3844 -a_nodata "0 0 0" -co compress=deflate -co tiled=yes landsat321Apuseni.vrt output/landsat321ApuseniDef.tif
	
	#~ compresie (!)lzw
	gdal_translate -a_srs EPSG:3844 -a_nodata "0 0 0" -co compress=lzw -co tiled=yes landsat321Apuseni.vrt output/landsat321ApuseniLzw.tif
	
	#~ compresie (!)jpeg
	gdal_translate -a_srs EPSG:3844 -a_nodata "0 0 0" -co compress=jpeg -co tiled=yes landsat321Apuseni.vrt output/landsat321ApuseniJpg.tif
	
	#~ compresie jpeg (!)ycbcr 
	gdal_translate -a_srs EPSG:3844 -a_nodata "0 0 0" -co compress=jpeg -co photometric=ycbcr -co tiled=yes landsat321Apuseni.vrt output/landsat321ApuseniJpgY.tif
	gdaladdo -r cubic --config COMPRESS_OVERVIEW JPEG --config PHOTOMETRIC_OVERVIEW YCBCR --config INTERLEAVE_OVERVIEW PIXEL output/landsat321ApuseniJpgY.tif 2 4 8 
		#~ (!)overviews
		
	#~ format webp standard
	gdal_translate -a_srs EPSG:3844 -of webp landsat321Apuseni.vrt output/landsat321Apuseni.webp
	ls -lh output/landsat321Apuseni*.* 
	
	#~ PAUZĂ: 10 min

#~ +++++++++++++++++++++++++++++++
#~ Partea a II-a. Geoprocesare DEM
#~ gdaldem - Tools to analyze and visualize DEMs.
	gdaldem
	#~ http://gdal.org/gdaldem.html
		
	#~ hillshade	
	gdaldem hillshade output/tm_srtm-L-34-079-s42.tif output/htm_srtm-L-34-079-s42.tif -co compress=deflate
	gdaldem hillshade output/tm_srtm-L-34-079-s42.tif output/htm_srtm-L-34-079-s42.tif -z 3  -az 130 -co compress=deflate
		#~ ne-am jucat puțin cu opțiunile -z -az
	
	cd output
	for i in srtm-L-34-0??-s42.tif; do gdaldem hillshade $i h$i -co compress=deflate -z 3; done
	cd..
	
	#~ slope
	gdaldem slope output/tm_srtm-L-34-079-s42.tif output/stm_srtm-L-34-079-s42.tif -co compress=deflate
	gdaldem slope output/tm_srtm-L-34-079-s42.tif output/stm_srtm-L-34-079-s421.tif -alg ZevenbergenThorne -co compress=deflate
		#~ detalii suplimentare http://www.unesco.org.uy/ci/fileadmin/phi/aqualac/GarciaRodriguez_et_al_p78-82.pdf
	gdaldem slope output/tm_srtm-L-34-079-s42.tif output/stm_srtm-L-34-079-s422.tif -p -co compress=deflate
		#~ -p use percent slope
	
	cd output
	for i in srtm-L-34-0??-s42.tif; do gdaldem slope $i s$i -p -co compress=deflate; done
	cd ..
	
	#~ detalii suplimetare:
	 #~ http://www.cartogis.org/docs/proceedings/2005/lee_clark.pdf

#~ ex. 2 Generati in mod batch o harta hipsometrica in culori (color-relief) pentru seturile srtm.
#~ ex. 3 Combinati intr-un mozaic optimizat toate careurile de harta hipsometrica. (atentie la compresie pentru DEM)
#~ (2+3, total 15 min)

	#~ PAUZĂ: 10 min


#~ +++++++++++++++++++++++++++++++
#~ Partea a III-a. Preprocesare scene satelitare
	#~ gdal-segment: https://github.com/cbalint13/gdal-segment
	gdal-segment -algo lsc ortofoto_dobrogea.tif -out output/ortofoto_lsc.shp
		#~ https://github.com/cbalint13/gdal-segment



#~ +++++++++++++++++++++++++++++++
#~ Partea a IV-a. Teledetecție: (!)pan-sharpening
#~ Următorul exercițiu este adaptat după: http://mapbox.com/blog/pansharpening-satellite-imagery-openstreetmap/
	gdalbuildvrt -separate -srcnodata "0 0 0" -vrtnodata "0 0 0" tm_landsat321-L-34-079.vrt B3-L-34-079.tif B2-L-34-079.tif B1-L-34-079.tif
	sed -i '6 i <ColorInterp>Red</ColorInterp>' tm_landsat321-L-34-079.vrt && sed -i '18 i <ColorInterp>Green</ColorInterp>' tm_landsat321-L-34-079.vrt && sed -i '30 i <ColorInterp>Blue</ColorInterp>' tm_landsat321-L-34-079.vrt
	
		#~ Comanda următoare execută pan-sharpening pentru tm_landsat321-L-34-079.vrt. Pentru a o executa e nevoie să instalați o bibliotecă suplimentară, denumită OTB. Pentru instalalre și alte detalii accesați: https://www.orfeo-toolbox.org/download/
	
	otbcli_BundleToPerfectSensor -ram 512 -inp BPan-L-34-079.tif -inxs tm_landsat321-L-34-079.vrt -out landsat321Pan-L-34-048.tif
	
	gdal_translate -ot Byte -scale 0 3000 0 255 -a_nodata "0 0 0" landsat321Pan-L-34-048.tif landsat321Pan-L-34-048-scalat.tif
	gdalwarp -r cubic -wm 4096 -multi -srcnodata "0 0 0" -dstnodata "0 0 0" -dstalpha   -wo OPTIMIZE_SIZE=TRUE -wo UNIFIED_SRC_NODATA=YES -t_srs EPSG:3844 -co tiled=yes -co compress=lzw landsat321Pan-L-34-048-scalat.tif landsat321Pan-L-34-048-s42.tif
```
