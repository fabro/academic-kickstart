+++
title = "Species Richness & Scales"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Distribuições, riqueza de especies e escalas

**Macroecologia**

Universidade Federal do Rio Grande do Sul
Instituto de Biociências
PPG-Ecologia


**NOTA**: Si no han cerrado R, seguimos con el mismo espacio de trabajo, pero si lo cerraron, tienen que cargar el "workspace" (archivo .RData) del ejercicio anterior pues tiene los objetos que vamos a usar en este ejercicio.

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(raster)
library(terra)
library(rnaturalearth)
library(sf)
```

En este ejemplo, vamos a usar datos de carnívoros para América del Sur, que están disponibles en la red en la base de la IUCN (http://www.iucnredlist.org/technical-documents/spatial-data) 
Uds podrían usar cualquier otro grupo de los disponíbles en la IUCN o en otros sitios (en este caso, tienen que bajar los "shapefiles", que son archivos con extensión '.shp' y los asociados)
```{r eval=FALSE}
amsur.carniv <- st_read("ejercicios_datos/amsur_carnivs.shp")
```

```{r eval=FALSE, echo=FALSE, results='hide',message=FALSE}
pdf("amsur_carniv.pdf")
plot(amsur.carniv["binomial"],col=rainbow(46, alpha=0.5))
dev.off()
```

```{r eval=FALSE}
plot(amsur.carniv["binomial"],col=rainbow(46, alpha=0.5))
```

"Proyectar" en coordenadas geográficas -- definir la proyección de los datos/shapefiles
```{r eval=FALSE}
st_crs(amsur.carniv) <- "+proj=longlat +datum=WGS84"
```

Generar un raster para América del Sur
```{r eval=FALSE}
amer_ras <- raster()
# Establecer el "extent" del raster para quedarnos justamente en las coordenadas de América del Sur
extent(amer_ras) <- c(-110,-29,-56,14)
```

Cambiar la resolución del raster de América del Sur para que sea de 1º long-lat.
```{r eval=FALSE}
res(amer_ras) <- 1
```

#Raster de riqueza de especies
Generar un raster para los datos de los carnivoros basado en el raster de América del Sur que ya tenemos, este nuevo raster será el mapa de riqueza de las especies de carnívoros de América del Sur!

**NOTA**: Hay otras formas de generar el raster de riqueza que son, en realidad, más apropiadas para hacer análisis biogeográficos/macroecológicos (Esta parte la vamos a ver más adelante)

```{r eval=FALSE}
carniv_raster <- terra::rasterize(amsur.carniv, amer_ras, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Mapa continental, países
```{r, eval=FALSE}
cont_map <- ne_countries(scale = "small")
```


Graficar el raster de los murciélagos. 
```{r eval=FALSE}
plot(carniv_raster)
plot(cont_map,add=T)
```
¿Qué tal, quedó bien?


#Dependencia de la escala
Generar otro raster de América del Sur con una resolución diferente, de 2 grados long-lat
```{r eval=FALSE}
amer_ras2 <- amer_ras
# definir la resolución de 2 grados
res(amer_ras2) <- 2
```

Generar el raster de los carnívoros para la nueva resolución (usando el nuevo raster de América del Sur en 2 grados long-lat)
```{r eval=FALSE}
carniv_raster2 <- terra::rasterize(amsur.carniv, amer_ras2, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar el nuevo raster 
```{r eval=FALSE}
plot(carniv_raster2)
plot(cont_map,add=T)
```
Y ahora, ¿Quedó bien? ¿igual que el anterior?

**NOTA**: Para mantener las diferentes gráficas de los mapas con diferentes resoluciones, tienen que abrir una nueva ventana de graficación en R. Vean el manual de R de cómo hacer eso. Si no, el gráfico generado va a aparecer en la misma "ventana" y van a "perder" el mapa anterior. Otra forma sería salvar (save as...) el gráfico y después abrir todos en otro programa para compararlos visualmente. De cualquier manera, más adelante veremos cómo graficarlos todos juntos.

Otra resolución, ¿más gruesa? Generar un raster de América del Sur con resolución de 4 grados long-lat
```{r eval=FALSE}
amer_ras4 <- amer_ras2

res(amer_ras4) <- 4
```

Generar el raster de los carnívoros con el nuevo de América del Sur en 4 grados long-lat
```{r eval=FALSE}
carniv_raster4 <- terra::rasterize(amsur.carniv, amer_ras4, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar 
```{r eval=FALSE}
plot(carniv_raster4)
plot(cont_map,add=T)
```
¿El patrón cambió?

¿Resolución más gruesa aún? Generar un nuevo raster de América del Sur a 8 grados long-lat
```{r eval=FALSE}
amer_ras8 <- amer_ras4

res(amer_ras8) <- 8
```

Y el raster de riqueza de los carnívoros en la nueva resolución
```{r eval=FALSE}
carniv_raster8 <- terra::rasterize(amsur.carniv, amer_ras8, field="binomial",fun=function(x,...){length(unique(na.omit(x)))})
```

Graficar 
```{r eval=FALSE}
plot(carniv_raster8)
plot(cont_map,add=T)
```
Y ahora, ¿cambió? ¿En dónde están las mayores diferencias?
```{r eval=FALSE}
par(mfrow=c(2,2))

plot(carniv_raster)
plot(cont_map,add=T)

plot(carniv_raster2)
plot(cont_map,add=T)

plot(carniv_raster4)
plot(cont_map,add=T)

plot(carniv_raster8)
plot(cont_map,add=T)

dev.off()
```


**NOTA:** Salven el "workspace" de R, es decir, todo lo que se hizo en este ejercicio debe quedar salvado como un archivo ".RData". Estos objetos serán usados en ejercicios posteriores. NO CIERREN R SIN HABER GUARDADO ESTE ARCHIVO!!!
