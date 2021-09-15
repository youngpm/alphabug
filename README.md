# Bilinear Resampling in Mapserver

Some examples odd image compositing with mapserver.

[data/east.tif](data/east.tif) and [data/west.tif](data/west.tif) are two 256x256 geotiffs in EPSG:4326, one covering the east and one covering the west hemispheres.  They do not overlap.  The are a solid red color, but with a alpha band going from 0 to 255, north to south.  

The expectation is that sampling a an image that spans the meridian will yield no artifacts; in particular if we upsample we still expect no compositing to occur.  

However, with bilinear upsampling, we see a pixel of overlap (at the native image's resolution) in the result.  This result is not present when one uses nearest neighbor resampling, or if we reference a VRT.  

You can recreate this with the following commands:

Composite both layers and the bilinear result has the artifact:
```
shp2img -m mapfile.map -o two-layers-bilinear.png -l "west-bilinear east-bilinear" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
shp2img -m mapfile.map -o two-layers-nearest.png -l "west-nearest east-nearest" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
```

Composite via a tile index and the bilinear result has the artifact:
```
shp2img -m mapfile.map -o east-west-tindex-bilinear.png -l "east-west-tindex-bilinear" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
shp2img -m mapfile.map -o east-west-tindex-nearest.png -l "east-west-tindex-nearest" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
```

Composite via a VRT and there is no artifact regardless of the resampling kernel:
```
shp2img -m mapfile.map -o east-west-vrt-bilinear.png -l "east-west-vrt-bilinear" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
shp2img -m mapfile.map -o east-west-vrt-nearest.png -l "east-west-vrt-nearest" -e -3.515625 -35.15625 3.515625 35.15625 -s 32 320
```

In QGIS, I have compared WMS of a raster that is bilinearly interpolated vs loading the raster in directly, and one observes that the WMS version is 1/2 a pixel inflated all the way around compared to the direclty loaded raster.  When one switches to nearest neighbor, the WMS agrees with the directly loaded raster.

I've obversed this behavior in mapserver 7.0.8 and 7.6.4.  
