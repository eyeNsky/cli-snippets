sometimes the knowing<br>
can not be found in what works<br>
but rather, what fails<br>

# How does that for loop work? sed what?
A couple of things I've found really useful. Mostly copy pastes from other sources that I've collected. I'll try to document the original reference if I have it. If you see something I've tweezed from you and would like credit let me know.

# grep magick to treat html like FTP for recursive download. wget grep
<pre><code>wget -q -O - http://oceandata.sci.gsfc.nasa.gov/MODISA/L2/2006/005/ |grep SST4|wget -N --wait=0.5 --random-wait --force-html -i - </pre></code>

# get list of NGS cors stations for a day into a list. grep pipe awk
<pre><code>wget ftp://geodesy.noaa.gov/cors/rinex/2010/365/ -q -O - | grep Directory | awk '{split($0,a,">"); split(a[2],b,"/");print b[1]}' >> cors.list</pre></code>

# File name, substitution 
get basename
<pre><code>for FILE in &#42;.tif;do echo ${FILE%.tif};done</pre></code>

string substitution-strip leading C
<pre><code>for FILE in &#42;.jpg;do mv $FILE ${FILE/C};done</pre></code>
replace leading C with something else
<pre><code>for FILE in &#42;.jpg;do echo ${FILE/C/new_};done</pre></code>
get basename
<pre><code>for TIF in &#42;/&#42;.tif;do echo ${TIF##&#42;/} ;done
n40w078/grdn40w078_1.tif grdn40w078_1.tif</pre></code>
replace space with underscore. Modified from:https://superuser.com/questions/678658/recursively-copy-and-rename-to-replace-spaces-with-underscore
<pre><code>for TIF in *.tif;do mv "$TIF" "$(echo $TIF| sed 's/ /_/g')";done</pre></code>
 
# USGS <br>
Make sure number of sub-directories == number of zip files

<pre><code>for ZIP in *.zip;do echo $ZIP;done | wc -l
503
for DIR in */grd*/;do echo $DIR;done | wc -l
503</pre></code>
IFSAR DEMs with bash
<pre><code>for ZIP in *.zip;do unzip $ZIP -d ${ZIP/.zip/};done
for DIR in */;do gdalwarp -t_srs EPSG:4326 -co TILED=YES $DIR*s**/ ../tifs/${DIR/\//}.tif;done</pre></code>

Had poor luck with the .img->.tif format in ossim tried this (the single quote is needed to keep the &&'s together):

<pre><code>ls */*.img | parallel -j 8 'gdalwarp -tr 0.00003 0.00003 {} ../tmp/{/.}.tif && gdal_translate -of AAIGrid ../tmp/{/.}.tif ../grd/{/.} && gdal_translate -co TILED=YES ../grd/{/.} ../AK_3m_NED/{/.}.tif && rm -r ../grd/{/.} ../tmp/{/.}.tif'</pre></code>

1/3 NED with parallel (note: could use gdal_translate; some directores are nested two deep, in this case they were in USGS_ prefixed dirs; also at least one of the directories contained a GeoTIFF rather that an ArcGrid...)
<pre><code>ls *.zip | parallel unzip {} -d{.}
for DIR in */grd*/;do echo $DIR;done | parallel -j 8 gdalwarp -t_srs EPSG:4326 -co TILED=YES {} ../tifs/{/.}.tif
# or if that doesn't work
for DIR in */grd*/;do echo $DIR;done | awk '{split($0,a,"\/");print"gdal_translate -a_srs EPSG:4326 -co TILED=YES", $0, "../tifs/"a[2]".tif"}' | parallel -j 16
# nested grd dir
for DIR in */USGS*/grd*/;do echo $DIR;done | parallel -j 8 gdalwarp -t_srs EPSG:4326 -co TILED=YES {} ../tifs/{/.}.tif
# or 
for DIR in */USGS*/grd*/;do echo $DIR;done | awk '{split($0,a,"\/");print"gdal_translate -a_srs EPSG:4326 -co TILED=YES", $0, "../tifs/"a[3]".tif"}' | parallel -j 16
</pre></code>
Parse out URL from the National Map CSV file to a file for wget, skipping the header.
<pre><code>awk 'NR>1 {split($0,a,",");print a[11]}' ned987_20171211_105245.csv > wget.list</pre></code>
Or one line to wget via parallel
<pre><code>awk 'NR>1 {split($0,a,",");print a[11]}' ned987_20171211_105245.csv | parallel -j 8 wget {}</pre></code>
S3 cli to poke around
<pre><code>aws s3 ls --no-sign-request 3://prd-tnm/StagedProducts/Elevation/ </pre></code>
Some of the TIFFs are COG (not sure if all are), so this returns a tileindex of the TIFFs in this dir:
<pre><code> aws s3 ls  --no-sign-request --recursive s3://prd-tnm/StagedProducts/Elevation/1m/Projects/TX_South_B8_2018/TIFF/ | grep -e ".tif" | awk '{split($0,a," ");print "/vsicurl/https://prd-tnm.s3.amazonaws.com/"a[4]}' | parallel -j 1 --progress gdaltindex .sqlite "{}" </pre></code>
...and the index contains the file URL (drop the /vsicurl/).
There is an index for the 1m project areas here:
<pre><code>http://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/1m/FullExtentSpatialMetadata/FESM_1m.gpkg</pre></code>
1/9th shapefile index:
<pre><code>http://prd-tnm.s3.amazonaws.com/index.html?prefix=StagedProducts/Elevation/19/FullExtentSpatialMetadata</pre></code>
1/3rd:
<pre><code>https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/13/FullExtentSpatialMetadata/FESM_13.gpkg</pre></code>
# ogr
<pre><code> for SHP in &#42;/roads.shp; do ogr2ogr -sql "SELECT &#42; FROM roads WHERE "type" LIKE 'motorway' OR "type" LIKE 'trunk' OR "type" LIKE 'primary' " -f SQLite -nln osm -append osm_thin.sqlite $SHP;done</pre></code>

# sed
<pre><code>sed -i '' 's|http://aws.com/|http://azure.net/|' *.HTM</code></pre>

# ogr and sed to work with ais
Based on: http://stackoverflow.com/questions/8914435/awk-sed-how-to-remove-parentheses-in-simple-text-file
This removes the parens, letters POINT, the triple space at the beginning, replaces the single space with a ',' and outputs lon,lat pairs -74.884883,44.973767
<pre><code> ogrinfo -sql "SELECT MMSI from Zone18_2014_08_Broadcast" Zone18_2014_08.gdb/ | grep POINT | sed 's/[()POINT]//g' | sed 's/   //g' | sed 's/ /,/g'</pre></code>

This will output x,y,z to a txt file
<pre><code>ogrinfo -sql "SELECT MMSI from Zone18_2014_08_Broadcast" Zone18_2014_08.gdb/ | grep POINT | sed 's/[(POINT]//g' | sed 's/)/,0/g' |sed 's/   //g' | sed 's/ /,/g' > out.txt</pre></code>

# s3cmd <br>
List Sentinel zip files at AWS
<pre><code>s3cmd ls s3://sentinel-s2-l1c/zips/</pre></code>

# az cli tools, bash, parallel, pipe
From inside a directory containing z,x,y map tile structure.<br>
All CAPS or %s are variables that would need to be passed to the cmd.
Upload tiles:
<pre><code> ls -1 -d */* | parallel --retries 3 --joblog tile-upload.log --progress -j %s "az storage blob upload-batch  --account-name %s --sas-token '%s' --content-cache-control 'public, max-age=%s' --max-connections 2 --destination CONTAINER/{} --source {} --validate-content" </code></pre>
Cache to CDN with wget calls:
<pre><code> for D in */*;do ls -1 $D/* >> ${D/\//_}.get;done && ls *.get | parallel wget -i {} -B https://CDN/CONTAINER/ -O /dev/null && rm *.get</code></pre>

# download, unzip and concatenate lidar tile index files
<pre><code>wget -q -O - https://coast.noaa.gov/htdata/lidar1_z/geoid12a/data/ | grep tileindex.zip | awk '{split($0,a,">Index SHP"); split(a[1],b,"href=");print b[8]}' | wget -i -

ls * | parallel unzip {}

for SHP in *.shp;do ogr2ogr -append -nln lidar ../lidar.shp $SHP;done
</pre></code>

# parse geom files for name, lat,lon, height
<pre><code>
 grep latlonh *.geom | awk '{split($0,a,":");print a[1]a[3]}'
 </pre></code>
 returns<br>C24581130.geom  28.987265723444441 -95.240654760067017 1108.390165457271678 WGE

# get min,max,average and count from geom files
Found part of this here: 
https://unix.stackexchange.com/questions/13731/is-there-a-way-to-get-the-min-max-median-and-average-of-a-list-of-numbers-in
<pre><code>
 grep meters_per_pixel_x *.geom | awk '{split($0,a," ");print a[2]}' | awk 'NR == 1 { max=$1; min=$1; sum=0 }                       
   { if ($1>max) max=$1; if ($1&ltmin) min=$1; sum+=$1;}
   END {printf "Min: %f\tMax: %f\tAverage: %f Count: %d\n", min, max, sum/NR, NR}'
    </pre></code>
# one line to embed lat,lon,height from geom into jpg
<pre><code>
grep latlonh *.geom | awk '{split($0,a,":");split(a[3],b," ");print "exiftool -q -overwrite_original_in_place -GPSLatitudeRef=N -GPSLatitude="b[1]" -GPSLongitudeRef=W -GPSLongitude="b[2]" -GPSAltitude="b[3]" "a[1] }'| sed -e 's|.geom|.jpg|' | parallel
</pre></code>

# two lines to get GPS hh:mm:ss from exif into an events file as seconds of week. need to know day of week to get the correct number of 86,400 offsets
<pre><code>
for JPG in *.jpg;do exif $JPG > ${JPG/.jpg/.txt};done
grep -e "GPS Time" *.txt | awk '{split($0,a,":");split(a[2],b,"|");printf "%s %f\n", a[1],b[2]*3600+a[3]*60+a[4]+3456
00}' > events.txt
</pre></code>
# export ossim height to env var
Found the last piece about sourcing the file here:https://stackoverflow.com/questions/11988663/awk-setting-environment-variables-directly-from-within-an-awk-script
<pre><code>
ossim-info --height 27.9 -97.5 -P /mnt/elevation/prefs | grep -e "Height above MSL:" | awk '{split($0,a,":");printf "export AGL='%d'", a[2]}'>agl.sh&& source agl.sh
</pre></code>

# need to combine this with the above to pipe the env var all the way from gpspipe. may need to install gawk... <br>
the grep buffer was killing me! thanks: http://blog.jpalardy.com/posts/grep-and-output-buffering/
also needed the fflush() from: https://unix.stackexchange.com/questions/33650/why-does-awk-do-full-buffering-when-reading-from-a-pipe
<pre><code>
gpspipe -w | grep --line-buffered lon | awk '{split($0,a,",");split(a[5],b,":");split(a[6],c,":");print "ossim-info --height " b[2], c[2]," -P /mnt/elevation/prefs";fflush()}' | parallel
</pre></code>
This almost does it:
<pre><code>
gpspipe -w | grep --line-buffered lon | awk '{split($0,a,",");split(a[5],b,":");split(a[6],c,":");print "ossim-info --height " b[2], c[2]," -P /mnt/elevation/prefs";fflush()}' | parallel | grep --line-buffered -e "Height above MSL:" | awk '{split($0,a,":");printf "export AGL='%d'\n", a[2];fflush()}'
</pre></code>
This does what I need it to do. I can't get anything past the > symbol, but don't need to. Just need one fix to set the base elevation. -m 1 in grep finds the first line with a 'lon' (longitude) and passes that down the pipe.
<pre><code>
gpspipe -w | grep -m 1 --line-buffered lon | awk '{split($0,a,",");split(a[5],b,":");split(a[6],c,":");print "ossim-info --height " b[2], c[2]," -P /mnt/elevation/prefs";fflush()}' | parallel | grep --line-buffered -e "Height above MSL:" | awk '{split($0,a,":");printf "export AGL=\"%d\"\n", a[2];fflush()}' > agl.sh && source agl.sh
</pre></code>

Stream elevations to a file that can be read by another program:
<pre><code>
gpspipe -w | grep --line-buffered lon | awk '{split($0,a,",");split(a[5],b,":");split(a[6],c,":");print "ossim-info --height " b[2], c[2]," -P /mnt/elevation/prefs";fflush()}' | parallel | grep --line-buffered -e "Height above MSL:" | awk '{split($0,a,":");printf "%d\n", a[2];fflush()}' > agl.txt
</pre></code>
Get elevations for lat/lon pairs in a CSV file where each point has a "P" in the name:
<pre><code>
grep --line-buffered P LLH_GNV.csv |awk '{split($0,a,",");print "ossim-info --height " a[2], a[3]," -P /mnt/elevation/prefs";fflush()}' | parallel | grep --line-buffered -e "Height above MSL:" | awk '{split($0,a,":");printf "%f\n", a[2];fflush()}'
</pre></code>
# Copy librarys from ldd to a directory
<pre><code>
ldd ./executable | grep -e "=>" | awk '{split($0,a," ");print "cp " a[3] " /out/path"}' | grep -e ".so" | bash
</pre></code>
# Make footprints from ortho imagery
<pre><code>ls *.tif | parallel --progress 'gdal_calc.py --quiet -A {}  --A_band 1 -B {} --B_band 2 -C {} --C_band 3 --calc="1*logical_and(A>0,B>0,C>0)" --outfile ../fp/{}'
cd ../fp/
ls *.tif | parallel --progress gdal_polygonize.py {} -f GML {.}.gml
for GML in *.gml;do ogr2ogr -where "DN=1" -f SQLITE -append out.sqlite $GML;done</pre></code>

Above does not add file name to the output. It only makes the footprints. This will add name into the fid col with a .1 or .xxx at the end.
Need to figure out the sql to make this cleaner...
<pre><code>
ls *.tif | parallel --progress 'gdal_calc.py --quiet -A {}  --A_band 1 -B {} --B_band 2 -C {} --C_band 3 --calc="1*logical_and(A>0,B>0,C>0)" --outfile ../fp/{}'
cd ../fp/
# the {.} at the end adds the tif name as the layer. End up looking like P15458929.1
ls *.tif | parallel --progress gdal_polygonize.py {} -f GML {.}.gml {.} 
# the -nln keeps ogr from writing separate layers for each fp
for GML in *.gml;do ogr2ogr -nln fp -where "DN=1" -f SQLITE -append out.sqlite $GML;done
</pre></code>
# Buffer and dissolve Sentinel Cloud mask
<pre><code>ogr2ogr -f SQLITE cloud-buffer.sqlite  L1C_T18SVE_A024032_20200128T154938_MSK_CLOUDS_B00.gml -dialect sqlite -sql "select ST_Union(ST_buffer(geometry, 5000)) as geometry FROM MaskFeature"</pre></code>
# zgrep Sentinal index to awk
<pre><code>zgrep -e "PASSED" ~/Downloads/index.csv.gz | awk '{split($0,a,",");print a[7]","a[10]","a[11]","a[12]","a[13]","a[14]}'</pre></code>
returns: CLOUD_COVER, NORTH_LAT,SOUTH_LAT,WEST_LON,EAST_LON,BASE_URL
<pre><code>0.0,44.2257183768,43.2356964818,29.1971656897,29.6265213541,gs://gcp-public-data-sentinel-2/tiles/35/T/PJ/S2A_MSIL1C_20150829T085006_N0204_R107_T35TPJ_20150829T085004.SAFE</pre></code>
# zgrep Sentinel index to get images from a particular tile
<pre><code>zgrep -e "18SVE" ~/Downloads/index.csv.gz > 18SVE.csv</pre></code>
# zgrep, grep, sort, head to get 100 images from a particular for 2019/2020 sorted by cloud
!zgrep -e "19TDF" index.csv.gz | grep -e "2020-\|2019" |   sort -n -k 7 -k 5 -t "," | head -n 100 

# OGR Examples
https://github.com/dwtkns/gdal-cheat-sheet
http://emapr.ceoas.oregonstate.edu/pages/education/how_to/how_to_ogr2ogr.html
https://www.bostongis.com/?content_name=ogr_cheatsheet#41

# Skip line-too-long pylint msg in Sublime
https://stackoverflow.com/questions/23500499/how-can-i-ignore-a-lint-error-for-a-line-with-sublime-text-3-anaconda
<pre><code>{ "pep8_ignore": ["E501"] }</pre></code>
# VirtualBox-Untested!!
Found this here:
https://superuser.com/questions/255270/how-to-copy-vhd-file-to-physical-hard-disk-using-dd-command

<pre><code>VBoxManage clonehd windows_xp64.VHDX --format RAW windows_xp64.RAW 
dd if=windows_xp64.RAW of=/dev/sdf</pre></code>
