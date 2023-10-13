+++
title = "Correlative approach"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Abordagem correlativa e escalas

**Macroecologia**

Universidade Federal do Rio Grande do Sul
Instituto de Biociências
PPG-Ecologia


**NOTA**: Tienen que cargar el "workspace" (archivo .RData) del ejercicio anterior pues este tiene los objetos que usaremos en este ejercicio.

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(raster)
library(maptools)
library(rgeos)
library(sp)
```

Cargar las variables ambientales
```{r eval=FALSE} 
aet <- raster("ejercicios_datos/AET.bil")

npp <- raster("ejercicios_datos/npp2.asc")

#Checar que los rasters estén "proyectados"
projection(npp)
projection(aet)

#Si no estuvieran proyectados, sin tener una referencia geográfica, hay que definir una (la "default", coordenadas geográficas)

proj4string(aet) <- CRS("+proj=longlat +ellps=WGS84 +towgs84=0,0,0,0,0,0,0 +no_defs")

proj4string(npp) <- CRS("+proj=longlat +ellps=WGS84 +towgs84=0,0,0,0,0,0,0 +no_defs")
```

Cortar los rasters de NPP e AET para el extent del raster de América del Sur (creado antes, de 1º de resolución)
```{r eval=FALSE}
aet_amer  <- crop(aet,extent(amer_ras))
npp_amer  <- crop(npp,extent(amer_ras))
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
carniv_ras_coords <- xyFromCell(carniv_raster, 1:length(values(carniv_raster)))
```
Obtener los valores de AET en los sitios en donde hay carnívoros. Cambiar NAs por 0s
```{r eval=FALSE}
carniv_ras_aet <- extract(aet_amer1_nas,carniv_ras_coords)
carniv_ras_rich <- values(carniv_raster)
carniv_ras_rich_nonas <- ifelse(is.na(carniv_ras_rich),0,carniv_ras_rich)
carniv_ras_aet_nonas <- ifelse(is.na(carniv_ras_aet),0,carniv_ras_aet)
```

#Enfoque Correlativo: riqueza ~ ambiente
Correlación entre riqueza de carnívoros y AET
```{r eval=FALSE}
cor(carniv_ras_aet_nonas, carniv_ras_rich_nonas)
cor.test(carniv_ras_aet_nonas, carniv_ras_rich_nonas)
```

#Considerando la autocorrelación espacial
```{r eval=FALSE}
library(SpatialPack)

modified.ttest(carniv_ras_aet_nonas, carniv_ras_rich_nonas, carniv_ras_coords)

```

Repetir las correlaciones (los pasos anteriores) para los patrones en diferentes escalas que generamos en el ejercicio anterior

¿Cómo se ven? 
¿Hay diferencias entre las escalas?
