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
 
# USGS IFSAR DEMs with bash
<pre><code> for ZIP in *.zip;do unzip $ZIP -d ${ZIP/.zip/};done
for DIR in */;do gdalwarp -t_srs EPSG:4326 -co TILED=YES $DIR*s**/ ../tifs/${DIR/\//}.tif;done</pre></code>

# ogr
<pre><code> for SHP in &#42;/roads.shp; do ogr2ogr -sql "SELECT &#42; FROM roads WHERE "type" LIKE 'motorway' OR "type" LIKE 'trunk' OR "type" LIKE 'primary' " -f SQLite -nln osm -append osm_thin.sqlite $SHP;done</pre></code>

# sed
<pre><code>sed -i '' 's|http://aws.com/|http://azure.net/|' *.HTM</code></pre>


# VirtualBox-Untested!!
Found this here:
https://superuser.com/questions/255270/how-to-copy-vhd-file-to-physical-hard-disk-using-dd-command

<pre><code>VBoxManage clonehd windows_xp64.VHDX --format RAW windows_xp64.RAW 
dd if=windows_xp64.RAW of=/dev/sdf</pre></code>
