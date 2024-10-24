+++
title = "Correlative approach and scale effects"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Enfoque correlativo y escalas

**Macroecología**

Instituto de Ecología, A.C. - INECOL


**NOTA**: Tienen que cargar el "workspace" (archivo .RData) del ejercicio anterior pues este tiene los objetos que usaremos en este ejercicio.

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(terra)
library(maptools)
library(rgeos)
library(sp)
```

Cargar las variables ambientales
```{r eval=FALSE} 
aet <- rast("ejercicios_datos/AET.bil")
npp <- rast("ejercicios_datos/npp2.asc")

#Checar que los rasters estén "proyectados"
crs(npp)
crs(aet)

#Si no estuvieran proyectados, sin tener una referencia geográfica, hay que definir una (la "default", coordenadas geográficas)

crs(aet) <- "epsg:4326"

crs(npp) <- "epsg:4326"

```

Cortar los rasters de NPP e AET para el extent del raster de América del Sur (creado antes, de 1º de resolución)
```{r eval=FALSE}
aet_amer  <- crop(aet,ext(amer_ras))
npp_amer  <- crop(npp,ext(amer_ras))
```

Agregar los valores de NPP y AET en una resolución mayor
```{r eval=FALSE}
aet_amer1  <- aggregate(aet_amer,2)
npp_amer1  <- aggregate(npp_amer,2)
```

Los valores de AET en los océanos es de 255, tienen que transformar para NA (este paso es sólo para AET, no para NPP)
```{r eval=FALSE}
aet_amer1_vals <- values(aet_amer1)
aet_amer1_vals <- ifelse(aet_amer1_vals==255,NA,aet_amer1_vals)
aet_amer1_nas <- aet_amer1
values(aet_amer1_nas) <- aet_amer1_vals
```

Obtener las coordenadas del raster de los carnívoros
```{r eval=FALSE}
bats_ras_coords <- xyFromCell(bats_raster, 1:length(values(bats_raster)))
```
Obtener los valores de AET en los sitios en donde hay carnívoros. Cambiar NAs por 0s
```{r eval=FALSE}
bats_ras_aet <- extract(aet_amer1_nas,bats_ras_coords)
bats_ras_rich <- values(bats_raster)
bats_ras_rich[is.na(bats_ras_rich)] <- 0
bats_ras_aet[is.na(bats_ras_aet)] <- 0
```

#Enfoque Correlativo: riqueza ~ ambiente
Correlación entre riqueza de carnívoros y AET
```{r eval=FALSE}
cor(bats_ras_aet, bats_ras_rich)
cor.test(bats_ras_aet, bats_ras_rich)
```

#Considerando la autocorrelación espacial
```{r eval=FALSE}
library(SpatialPack)

modified.ttest(bats_ras_aet_nonas, bats_ras_rich_nonas, bats_ras_coords)

```

Repetir las correlaciones (los pasos anteriores) para los patrones en diferentes escalas que generamos en el ejercicio anterior

¿Cómo se ven? 
¿Hay diferencias entre las escalas?
