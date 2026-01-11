+++
title = "Species richness and spatial scales"

# Authors. Comma separated list.
authors = ["Fabricio Villalobos"]

  
+++

# Geographic biodiversity patterns and spatial scales

**Macroecology**

Julius-Maximilians-Universität Würzburg


Load the following:
```{r eval=FALSE}
library(terra)
library(rnaturalearth)
library(sf)
```

In this example, we'll use range data for bats of tropical America that were downloaded from the IUCN website (https://www.iucnredlist.org/resources/spatial-data-download).
```{r eval=FALSE}
bats <- st_read("exercises_data/phyllostomidae_shapes/phylllostomidae_marcelo.shp")
```

```{r eval=FALSE, echo=FALSE, results='hide',message=FALSE}
pdf("bats.pdf")
plot(bats["binomial"],col=rainbow(46, alpha=0.5))
dev.off()
```

```{r eval=FALSE}
plot(bats["binomial"],col=rainbow(46, alpha=0.5))
```

"Project" in geographgic coordinates -- define the projection of the data (shapefiles)
```{r eval=FALSE}
st_crs(bats) <- "+proj=longlat +datum=WGS84"
```

Create a raster for the Americas
```{r eval=FALSE}
amer_ras <- rast()
# Set the raster extent to keep just the part corresponding to the Americas
ext(amer_ras) <- c(-150,-20,-60,40)
```

Change the spatial resolution to de 1º long-lat.
```{r eval=FALSE}
res(amer_ras) <- 1
```

#Species richness raster
Create a raster of species richness across the Americas

**NOTE**: This is just a "quick and dirty" way of doing it. There are other, more correct ways of doing so (e.g., with the letsR package)

```{r eval=FALSE}
bats_rast <- terra::rasterize(bats, amer_ras, field="binomial",
                fun="count",na.rm=TRUE)
                
writeRaster(bats_rast,"bats_rast.tif")

```

Continental map, with countries
```{r, eval=FALSE}
cont_map <- ne_countries(scale = "small", returnclass="sv")
```


Plot the bat richness raster
```{r eval=FALSE}
plot(bats_rast)
plot(cont_map,add=T)
```
How is it?


#Scale depence
Create another raster, now with a different resolution: 2 degrees long-lat
```{r eval=FALSE}
amer_ras2 <- amer_ras
# define the new resolution
res(amer_ras2) <- 2
```

Create the richness raster for the new resolution
```{r eval=FALSE}
bats_rast2 <- terra::rasterize(bats, amer_ras2, field="binomial",fun="count")

writeRaster(bats_rast2,"bats_rast2.tif")
```

Plot the new raster
```{r eval=FALSE}
plot(bats_rast2)
plot(cont_map,add=T)
```
and now, Is it ok? Same as the previous one?

Other solution, larger one: 4 degrees long-lat
```{r eval=FALSE}
amer_ras4 <- amer_ras2

res(amer_ras4) <- 4
```

Create the raster at 4 degrees
```{r eval=FALSE}
bats_rast4 <- terra::rasterize(bats, amer_ras4, field="binomial",fun="count")

writeRaster(bats_rast4,"bats_rast4.tif")
```

Plot the raster
```{r eval=FALSE}
plot(bats_rast4)
plot(cont_map,add=T)
```
Did the pattern change?

Larger resolution, let's go for 8 degrees
```{r eval=FALSE}
amer_ras8 <- amer_ras4

res(amer_ras8) <- 8
```

and the raster at the largest resolution
```{r eval=FALSE}
bats_rast8 <- terra::rasterize(bats, amer_ras8, field="binomial",fun="count")

writeRaster(bats_rast8,"bats_rast8.tif")
```

Plot the raster
```{r eval=FALSE}
plot(bats_rast8)
plot(cont_map,add=T)
```
and what now? did it change? Where are the major differences?
```{r eval=FALSE}
par(mfrow=c(2,2))

plot(bats_rast)
plot(cont_map,add=T)

plot(bats_rast2)
plot(cont_map,add=T)

plot(bats_rast4)
plot(cont_map,add=T)

plot(bats_rast8)
plot(cont_map,add=T)

dev.off()
```

Just to check and confirm the "good" use of these data for appropriate macroecological/biogeographical analyses, let's create a raster with an appropriate package (letsR - shameless selfpromotion!)
```{r eval=FALSE}
bats8 <- lets.presab(bats, resol = 8, count = T)
```

**NOTE:** Save the workspace of R, just in case we use the created objects later
