+++
title = "Correlative approach and spatial scales"

# Authors. Comma separated list.
authors = ["Fabricio Villalobos"]

  
+++

# Correlative approach and spatial scales

**Macroecology**

Julius-Maximilians-Universität Würzburg


**NOTE**: Load the workspace (RData file) from the previous exercise. It has the objects for this exercise

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(terra)
library(sp)
```

Cargar las variables ambientales
```{r eval=FALSE} 
aet <- rast("exercises_data/AET.bil")


#Check if they are "projected"
crs(aet)

#If they were not, without a geographic reference, we have to define it (the default, geographic coordinates)

crs(aet) <- "epsg:4326"

```

Crop the raster of AET to the exten of our study domain (the Americas, created before at 1º resolution)
```{r eval=FALSE}
aet_amer  <- crop(aet,ext(amer_ras))
```

Aggregate the values to a larger resolution
```{r eval=FALSE}
aet_amer1  <- aggregate(aet_amer,2)
```

AET values in the ocean are 255, we need to transform them to NA
```{r eval=FALSE}
aet_amer1_vals <- values(aet_amer1)
aet_amer1_vals <- ifelse(aet_amer1_vals==255,NA,aet_amer1_vals)
aet_amer1_nas <- aet_amer1
values(aet_amer1_nas) <- aet_amer1_vals
```

Get the coordinates for the bat species richness raster
```{r eval=FALSE}
bats_ras_coords <- xyFromCell(bats_rast, 1:length(values(bats_rast)))
```
Get the values of AET for the sites (gridcells) where we have bats. Change NAs for 0s
```{r eval=FALSE}
bats_ras_aet <- extract(aet_amer1_nas,bats_ras_coords)
bats_ras_rich <- values(bats_rast)
bats_ras_rich[is.na(bats_ras_rich)] <- 0
bats_ras_aet[is.na(bats_ras_aet)] <- 0
```

#Correlative approach: spp richness ~ environment
Correlation between bat species richness and AET
```{r eval=FALSE}
cor(bats_ras_aet[,1], bats_ras_rich)
cor.test(bats_ras_aet[,1], bats_ras_rich)
```

#Consider spatial autocorrelation
```{r eval=FALSE}
library(SpatialPack)

modified.ttest(bats_ras_aet[,1], bats_ras_rich, bats_ras_coords)

```

Repeat the correlations for the different resolutions of our previous exercise

How do they look?
Are there any differences among scales? Which ones? 
