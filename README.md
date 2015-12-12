# cli-snippets
One liners I always need but can't remember...
# How does that for loop work? sed what?
A couple of things I've found really useful. Mostly copy pastes from other sources that I've collected. I'll try to document the original reference if I have it. If you see something I've tweezed from you and would like credit let me know.

# grep magick to treat html like FTP for recursive download. wget grep
wget -q -O - http://oceandata.sci.gsfc.nasa.gov/MODISA/L2/2006/005/ |grep SST4|wget -N --wait=0.5 --random-wait --force-html -i -

# get list of NGS cors stations for a day into a list. grep pipe awk
wget ftp://geodesy.noaa.gov/cors/rinex/2010/365/ -q -O - | grep Directory | awk '{split($0,a,">"); split(a[2],b,"/");print b[1]}' >> cors.list
