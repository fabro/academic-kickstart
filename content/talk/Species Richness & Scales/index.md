+++
title = "Species Richness & Scales”

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++


# Polígonos, riqueza y escalas

**Macroecología**

Posgrado en Ciencias

Instituto de Ecología, A.C.


**NOTA**: Si no han cerrado R, seguimos con el mismo espacio de trabajo, pero si lo cerraron, tienen que cargar el "workspace" (archivo .RData) del ejercicio anterior pues tiene los objetos que vamos a usar en este ejercicio.

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(raster)
library(terra)
library(rnaturalearth)
library(sf)
```

En este ejemplo, vamos a usar datos de murciélagos, que están disponibles en la red en la base de la IUCN (http://www.iucnredlist.org/technical-documents/spatial-data) 
Uds podrían usar cualquier otro grupo de los disponíbles en la IUCN o en otros sitios (en este caso, tienen que bajar los "shapefiles", que son archivos con extensão '.shp' y los asociados)
```{r eval=FALSE}
bats <- st_read("ejercicios_datos/phyllostomidae_shapes/phylllostomidae_marcelo.shp")
```

```{r eval=FALSE, echo=FALSE, results='hide',message=FALSE}
pdf("bats.pdf")
plot(bats["binomial"],col=rainbow(46, alpha=0.5))
dev.off()
```
Graficar las distribuciones
```{r eval=FALSE}
plot(bats["binomial"],col=rainbow(46, alpha=0.5),
     main="murciélagos")
```

Revisemos qué sistema de coordenadas (coordinate reference system; CRS) tiene el objeto `bats`
```{r eval=FALSE}
st_crs(bats)
```

Generar un raster para América
```{r eval=FALSE}
amer_ras <- raster::raster()
# Establecer el "extent" del raster para quedarnos justamente en las coordenadas de América
extent(amer_ras) <- c(-150,-20,-60,40)
```

Si ven el objeto generado, este se tiene que ver así:
```{r eval=FALSE}
class      : RasterLayer 
dimensions : 180, 360, 64800  (nrow, ncol, ncell)
resolution : 0.3611111, 0.5555556  (x, y)
extent     : -150, -20, -60, 40  (xmin, xmax, ymin, ymax)
crs        : +proj=longlat +datum=WGS84 +no_defs 
```

Cambiar la resolución del raster de América para que sea de 1º long-lat.
```{r eval=FALSE}
res(amer_ras) <- 1
```

Ahora tendría que verse así
```{r eval=FALSE}
class      : RasterLayer 
dimensions : 100, 130, 13000  (nrow, ncol, ncell)
resolution : 1, 1  (x, y)
extent     : -150, -20, -60, 40  (xmin, xmax, ymin, ymax)
crs        : +proj=longlat +datum=WGS84 +no_defs 
```

#Raster de riqueza de especies
Generar un raster para los datos de los murciélagos basado en el raster de América que ya tenemos, este nuevo raster será el mapa de riqueza de las especies de murciélagos de América!

**NOTA**: Hay otras formas de generar el raster de riqueza que son, en realidad, más apropiadas para hacer análisis biogeográficos/macroecológicos (Esta parte la vamos a ver más adelante)

```{r eval=FALSE}
bats_raster <- terra::rasterize(bats, amer_ras, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})

```

Mapa continental, países
```{r, eval=FALSE}
cont_map <- ne_countries(scale = "small")
```


Graficar el raster de los murciélagos. 
```{r eval=FALSE}
plot(bats_raster)
plot(cont_map,add=T)
```
¿Qué tal, quedó bien?

# Dependencia de la escala
Generar otro raster de América con una resolución diferente, de 2 grados long-lat
```{r eval=FALSE}
amer_ras2 <- amer_ras
# definir la resolución de 2 grados
res(amer_ras2) <- 2
# el "objeto" tendría que verse así:
amer_ras2
class      : RasterLayer 
dimensions : 50, 65, 3250  (nrow, ncol, ncell)
resolution : 2, 2  (x, y)
extent     : -150, -20, -60, 40  (xmin, xmax, ymin, ymax)
crs        : +proj=longlat +datum=WGS84 +no_defs 
```

Generar el raster de los murciélagos para la nueva resolución (usando el nuevo raster de América en 2 grados long-lat)
```{r eval=FALSE}
bats_raster2 <- rasterize(bats, amer_ras2, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar el nuevo raster 
```{r eval=FALSE}
plot(bats_raster2)
plot(cont_map,add=T)
```
Y ahora, ¿Quedó bien? ¿igual que el anterior?

**NOTA**: Al usar R sin un IDE, para mantener las diferentes gráficas de los mapas con diferentes resoluciones, tienen que abrir una nueva ventana de graficación en R. Vean el manual de R de cómo hacer eso. Si no, el gráfico generado va a aparecer en la misma "ventana" y van a "perder" el mapa anterior. Otra forma sería salvar (save as...) el gráfico y después abrir todos en otro programa para compararlos visualmente. De cualquier manera, más adelante veremos cómo graficarlos todos juntos.

Otra resolución, ¿más gruesa? Generar un raster de América con resolución de 4 grados long-lat
```{r eval=FALSE}
amer_ras4 <- amer_ras2
res(amer_ras4) <- 4
```

Generar el raster de los murciélagos con el nuevo de América en 4 grados long-lat
```{r eval=FALSE}
bats_raster4 <- rasterize(bats, amer_ras4, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar 
```{r eval=FALSE}
plot(bats_raster4)
plot(cont_map,add=T)
```
¿El patrón cambió?

¿Resolución más gruesa aún? Generar el nuevo raster de América a 8 grados long-lat
```{r eval=FALSE}
amer_ras8 <- amer_ras4
res(amer_ras8) <- 8
```

Y el raster de riqueza de los murciélagos en la nueva resolución
```{r eval=FALSE}
bats_raster8 <- rasterize(bats, amer_ras8, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar 
```{r eval=FALSE}
plot(bats_raster8)
plot(cont_map,add=T)
```
Y ahora, ¿cambió? ¿En dónde están las mayores diferencias?
```{r eval=FALSE}
par(mfrow=c(2,2))

plot(bats_raster)
plot(cont_map,add=T)

plot(bats_raster2)
plot(cont_map,add=T)

plot(bats_raster4)
plot(cont_map,add=T)

plot(bats_raster8)
plot(cont_map,add=T)

dev.off()
```


## Ahora para México!

Generar un nuevo raster (vacio)
```{r eval=FALSE}
ras_2 <- raster()

#definir valores de cero para el raster
values(ras_2) <- 0
# definir el "extent" para que corresponda con las coordenadas de México. ¿Cuáles son?
extent(ras_2) <- c(-130,-80,14,35)
# definir la resolución
res(ras_2) <- 1
```

Generar el raster de México usando el raster generado antes cortado con el mapa de México que podemos generar 

```{r eval=FALSE}
mex_map <- ne_countries(scale = 'small', country = "Mexico",returnclass = "sf")
ras_mex <- rasterize(mex_map,ras_2)
```

Graficar el nuevo raster para México
```{r eval=FALSE}
plot(ras_mex)
plot(as(mex_map,"Spatial"), add=T)
```
¿Quedó bien?

Crear el raster de riqueza de los murciélagos de México usando el raster del país
```{r eval=FALSE}
bats_mexraster <- rasterize(bats, ras_mex, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar el resultado. ¿Cómo está distribuída la riqueza de murciélagos en México? ¿Dónde hay más/menos especies?
```{r eval=FALSE}
#Gerar una "máscara" para que el raster quede justamente dentro de México
batsmex_forma <- mask(bats_mexraster, mex_map)

plot(batsmex_forma)
plot(as(mex_map,"Spatial"), add=T)
```

Ahora con una resolución más gruesa (2 grados long-lat)
```{r eval=FALSE}
ras_mex2 <- ras_mex

res(ras_mex2) <- 2
```

raster de riqueza de murcélagos
```{r eval=FALSE}
bats_mexraster2 <- rasterize(bats, ras_mex2, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar
```{r eval=FALSE}
batsmex_forma2 <- mask(bats_mexraster2, mex_map)

plot(batsmex_forma2)
plot(as(mex_map,"Spatial"), add=T)
```
¿Quedó bien? ¿Cambió?

Resolución más gruesa aún
```{r eval=FALSE}
ras_mex5 <- ras_mex2
res(ras_mex5) <- 5
```

Raster de riqueza de murciélagos
```{r eval=FALSE}
bats_mexraster5 <- rasterize(bats, ras_mex5, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar. 
```{r eval=FALSE}
batsmex_forma5 <- mask(bats_mexraster5, mex_map)

plot(batsmex_forma5)
plot(as(mex_map,"Spatial"), add=T)
```
¿Cambió? ¿Cuánto? ¿Dónde hay más diferencias?

```{r eval=FALSE}
par(mfrow=c(1,3))

plot(batsmex_forma)
plot(as(mex_map,"Spatial"), add=T)

plot(batsmex_forma2)
plot(as(mex_map,"Spatial"), add=T)

plot(batsmex_forma5)
plot(as(mex_map,"Spatial"), add=T)

dev.off()
```

