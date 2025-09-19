+++
title = "Enfoque correlativo y efecto de escala"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Enfoque correlativo y escalas

**Macroecología**

Universidad de Antioquia - Posgrado de Biología


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
amer_ras <- rast()
# Establecer el "extent" del raster para quedarnos justamente en las coordenadas de América del Sur
ext(amer_ras) <- c(-110,-29,-56,14)
res(amer_ras) <- 1

aet_amer  <- crop(aet[[1]],ext(amer_ras))
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
carniv.rast <- rast("carniv.rast.tif")

carniv_ras_coords <- xyFromCell(carniv.rast, 1:length(values(carniv.rast)))
```
Obtener los valores de AET en los sitios en donde hay carnívoros. Cambiar NAs por 0s
```{r eval=FALSE}
carniv_ras_aet <- extract(aet_amer1_nas,carniv_ras_coords)
carniv_ras_rich <- values(carniv.rast)
carniv_ras_rich[is.na(carniv_ras_rich)] <- 0
carniv_ras_aet[is.na(carniv_ras_aet)] <- 0
```

#Enfoque Correlativo: riqueza ~ ambiente
Correlación entre riqueza de carnívoros y AET
```{r eval=FALSE}
cor(carniv_ras_aet, carniv_ras_rich)
cor.test(carniv_ras_aet[,1], carniv_ras_rich)
```

#Considerando la autocorrelación espacial
```{r eval=FALSE}
library(SpatialPack)

modified.ttest(carniv_ras_aet[,1], carniv_ras_rich, carniv_ras_coords)

```

Repetir las correlaciones (los pasos anteriores) para los patrones en diferentes escalas que generamos en el ejercicio anterior

¿Cómo se ven? 
¿Hay diferencias entre las escalas?
