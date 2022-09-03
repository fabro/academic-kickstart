+++
title = "Species Richness & Scales"
time_start = 2022-09-05T09:00:00
# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++
---
"Presencias y distribuciones geográficas de las especies"

**Macroecología**
Posgrado en Ciencias
Instituto de Ecología, A.C.
---

##### parte del código está basado en la vignette del paquete `dismo` (Hijmans & Elith 2011) y código de Vijay Barve (https://vijaybarve.wordpress.com) para mapear con ggplot2

Ya vimos de dónde vienen los datos primarios de biodiversidad (e.g. datos de presencia; almacenados en museos, colecciones científicas). Muchos de esos datos ahora están disponibles en internet!!!

# Ejercicio 1

Cargar los paquetes (TODOS!) necesarios para este ejercicio (estos ya deben estar instalados en su computadora)

```{r eval=FALSE}
library(sf)
library(terra)
library(maps)
library(dismo) 
library(rgdal)
library(rnaturalearth)
library(tidyterra)
library(ggplot2)
library(rgbif)
library(dplyr)
```

Obtener datos de presencia de una especie cualquiera a partir de la base de datos GBIF (data.gbif.org) directamente desde R!
**NOTA**: Necesitan estar conectados a internet. Este proceso puede tardar, dependiendo de la especie, si esta tiene (o no) muchos registros

```{r eval=FALSE}
sp_1 <- occ_search(scientificName = "Glossophaga morenoi")
```
**NOTA**: Uds tienen que colocar el nombre de la especie deseada.

El objeto `sp_1` es una lista con datos sobre los resultados obtenidos en GBIF, para trabjar únicamente con la tabla de registros hay que seleccionar el objeto _data_ dentro del mismo
```{r eval=FALSE}
sp_1 <- sp_1$data
```

Ver las dimensiones del objeto generado (i.e. cuántas filas y cuántas columnas tiene)
```{r eval=FALSE}
dim(sp_1)
```
Checar el nombre de las columnas (para después buscar únicamente las de posición geográfica: lat/long)
```{r eval=FALSE}
names(sp_1)
```

Crear otro objeto a partir del anterior para quedarse únicamente con long/lat

```{r eval=FALSE}
sp1_points <- select(sp_1,decimalLongitude,decimalLatitude)
```
**NOTA**: el nombre de la variable puede ser diferente (.e.g "LATITUDE", "Latidude", "lat", etc. Siempre hay que checar antes)

Graficar esos puntos (x,y)
```{r eval=FALSE}
ggplot(sp1_points)+
 geom_point(aes(decimalLongitude,decimalLatitude),
            col="blue",pch=19)
```
Agregar el mapa del mundo. Para eso, necesitamos cargar el mapa (que viene con el paquete "rnaturalearth")
```{r eval=FALSE}
wrld <- ne_countries(scale = "small",returnclass = "sf") # división política 

```

¿Hay algún dato extraño?
¿Notaron alguna advertencia, por qué salió?

Dibujando por capas, primero la división política y luego los puntos. Para dibujar datos de objetos diferentes en `ggplot`, pasamos el nombre del objeto al argumento `data` de cada capa que queremos graficar.

Graficar primero el mundo y después los puntos
```{r eval=FALSE}
ggplot()+
 geom_sf(data=wrld)
```
Agregar los puntos
```{r eval=FALSE}
ggplot()+
 geom_sf(data=wrld)+
 geom_point(data=sp1_points,aes(decimalLongitude,decimalLatitude),
            col="blue",pch=19,size=1)
```
**NOTA**: los argumentos "size" y "pch" simplemente definen el tamaño y tipo de los puntitos. Pueden jugar con eso y encontrar el que les parezca mejor.

Dependiendo de la especie, el mapa puede ser muy grande, ¿cierto? Da para generar un mapa "menor", sabiendo en dónde están nuestros puntos.
En ese caso, vamos a crear un mapa sólo para México (o para cualquer país, sólo hay que cambier el "NAME")
```{r eval=FALSE}
mex_map <- filter(wrld,name=="Mexico")
```
Ahora pueden repetir los pasos anteriores y mapear únicamente para México (o el país que hayan escogido).

```{r eval =FALSE}
ggplot()+
 geom_sf(data=mex_map)+
 geom_point(data=sp1_points,aes(decimalLongitude,decimalLatitude),
            col="blue",pch=19,size=1)
```

# Data checking & cleaning 

Checar si hay "duplicados"
```{r eval=FALSE}
sp1_dups <- duplicated(sp1_points)
### NOTA: la función "duplicated" retorna el resultado de una prueba lógica (e.g. TRUE or FALSE)
# ¿cuántos son duplicados?
length(which(sp1_dups==TRUE))
# ¿cuántos NO son duplicados?
length(which(sp1_dups==FALSE))
# Quedarse sólo con los registros únicos (respecto a la combinación long/lat)
sp1_nodups <- distinct(sp1_points)
# ¿Cuáles son las dimensiones de ese objeto nuevo?
dim(sp1_nodups)
# Ver los primero elementos/datos del nuevo
head(sp1_nodups)
```

Checar si hay datos con longitud/latitud cero
```{r eval=FALSE}
#longitud
filter(sp1_nodups,decimalLongitude==0)
#latitud
filter(sp1_nodups,decimalLatitude==0)
```
**NOTA**: Acordarse que los nombres pueden ser diferentes (Latitude, latitude, lat, etc...)
Usamos la función `filter` para buscar filas que cumplan con nuestra condición (ej: que los valores para la longitud o latitud sean iguales a 0).

Checar que no haya datos con NAs
```{r eval=FALSE}
filter(sp1_nodups,is.na(decimalLatitude))
filter(sp1_nodups,is.na(decimalLongitude))
# eliminar registros sin datos
sp1_points_nonas  <- na.omit(sp1_nodups)
```
Usamos `is.na` para revisar si un elemento es NA o no. Noten que al haber eliminado los registros duplicados primero, solo queda una fila con NAs. ¿Cuántos registros eran NA inicialmente?


# Ejercicio 2

Graficar con otros mapas, usando otros datos (e.g. bajar para otra espécie).

```{r eval=FALSE}
jaguarundi <- occ_data(scientificName = 'Herpailurus yagouaroundi', hasCoordinate = TRUE)
```
**NOTA**: Uds pueden escoger la especie que quieran.

Obtener el mapa del mundo
```{r eval=FALSE}
world = ne_countries(scale = "small", returnclass = "sf")
```

Graficar los puntos usando el mapa como poligono y definiendo características a los puntos (color, tamaño, etc)
```{r eval=FALSE}
ggplot(world) +
  geom_sf(fill = "white", # relleno de los países
          color = "gray40", # color de los bordes de los polígonos
          size = .2) + # ancho de los bordes
      geom_jitter(data = jaguarundi$data, # jitter mueve los puntos para que no se encimen
    aes(x=jaguarundi$data$decimalLongitude, 
        y=jaguarundi$data$decimalLatitude), 
    alpha=0.6, #transparencia de los puntos
             size = 4, color = "red") +
labs(title = "Herpailurus yagouarundi")

```
> Cuando el argumento "_size_" para el ancho de las líneas cambie su funcionamiento en futuras versiones de ggplot2, usar `linewidth`

¿Quedó bien? ¿y usando otra especie?


# Ejercicio 3

## Range maps from point data
En esta parte, se usaran funciones para generar mapas "simples" basados en la geometría (e.g. distancias entre puntos, etc.). NO consideran variables ambientales!

## Convex hull 
Este modelo crea un polígono convexo (envoltura) alrededor de los puntos de presencia.
```{r eval=FALSE}
#crear un raster que va a usarse como "máscara"
r <- raster()
values(r) <- 1
ext <- c(-130,-80,14,30)
r1 <- crop(r,ext)

sp_hull <- convHull(sp1_points_nonas)
sp_hullmodel <- predict(sp_hull, r1)
plot(sp_hullmodel, main='Convex Hull')
plot(as(wrld,"Spatial"), add=TRUE) 
points(sp1_points_nonas, pch='+')

```

## Círculos 
Este modelo dibuja círculos alrededor de los puntos de presencia.
```{r eval=FALSE}
sp_circ <- circles(sp1_points_nonas, lonlat=TRUE)
sp_circles <- predict(r1, sp_circ, mask=TRUE)

plot(sp_circles, main='Circles')
points(sp1_points_nonas, pch=19, cex=0.5, col="red")

#Jugar con la distancia usada para diseñar los "circles"
sp_circ2 <- circles(sp1_points_nonas, d=100000, lonlat = TRUE)
sp_circles2 <- predict(r1, sp_circ2, mask=TRUE)

plot(sp_circles2, main='Circles')
points(sp1_points_nonas, pch=19, cex=0.5, col="red")
```

# Ejercicio 5

Ahora, vamos a incluir datos ambientales/climáticos

Bajar los datos de clima del site www.worldclim.org (esto puede tardarse un poco, un par de minutos)
```{r eval=FALSE}
world_bioclim<-getData("worldclim", var = "bio", res = 10)
```

```{r eval=FALSE, echo=FALSE, results='hide',message=FALSE}
world_bioclim <- getData(name='worldclim',res=10,var='bio',path="/Users/fabro/laboratorio Macroecología/Cursos/posgrado_INECOL/Macroecologia/",download = F)
```

Estas variables están todas "juntas". Vamos a separarlas y dejar un objeto a parte que se puede usar para agarrar cada una de las variables separadas una por una
```{r eval=FALSE}
world_bioclim_inds<-unstack(world_bioclim)
```


### clave de las variables
```{r eval=FALSE}
BIO1 = Annual Mean Temperature
BIO2 = Mean Diurnal Range (Mean of monthly (max temp - min temp))
BIO3 = Isothermality (BIO2/BIO7) (* 100)
BIO4 = Temperature Seasonality (standard deviation *100)
BIO5 = Max Temperature of Warmest Month
BIO6 = Min Temperature of Coldest Month
BIO7 = Temperature Annual Range (BIO5-BIO6)
BIO8 = Mean Temperature of Wettest Quarter
BIO9 = Mean Temperature of Driest Quarter
BIO10 = Mean Temperature of Warmest Quarter
BIO11 = Mean Temperature of Coldest Quarter
BIO12 = Annual Precipitation
BIO13 = Precipitation of Wettest Month
BIO14 = Precipitation of Driest Month
BIO15 = Precipitation Seasonality (Coefficient of Variation)
BIO16 = Precipitation of Wettest Quarter
BIO17 = Precipitation of Driest Quarter
BIO18 = Precipitation of Warmest Quarter
BIO19 = Precipitation of Coldest Quarter
```

Graficar una de las variables
```{r eval=FALSE}
plot(world_bioclim_inds[[1]], zlim=c(-270,320), axes = F, box=F, col=rev(heat.colors(25)), ext = matrix(c(180,90,-180,-90), nrow = 2))
```
O un plot más "fancy"
```{r eval=FALSE}
bio1 <- as(world_bioclim_inds[[1]],"SpatRaster")
ggplot()+ 
     geom_spatraster(data=bio1,aes(fill=bio1)) + scale_fill_gradientn(colours=rev(c("brown","red","yellow","darkgreen","green")))
```


Grafiquen algunas de esas variables para América y para México. Para eso, hay que modificar el "ext=matrix(c(XX,XX,XX,XX)))" de la función anterior, colocando las coordenadas correctas para América y para México (Por separado, o sea, un mapa para cada una)


# Ejercicio 6
##Ecological Niche Model (Bioclim)

Vamos a quedarnos únicamente con dos variables
```{r eval=FALSE}
bio1<- world_bioclim_inds[[1]]
bio12<- world_bioclim_inds[[12]]
env.variables<- stack (bio1, bio12)
```

Establecer el "extent" de México e cortar las variables para ese extent
```{r eval=FALSE}
mex.extent <- c(-130,-80,14,30)
mex.env <- crop(env.variables,mex.extent)
```

Generar el modelo Bioclim (Ecological Niche Model: ENM)
```{r eval=FALSE}
# Convertir los puntos de la especie en un objeto tipo `matrix` 
sp1_points_mat <- as.matrix(sp1_points_nonas)

# Correr el ENM (Bioclim)
sp1_bioclim <- bioclim(mex.env, sp1_points_mat)
plot(sp1_bioclim, a=1, b=2, p=0.85)

# Generar una predicción del área adecuada ("suitable") basada en el ENM
sp1.map<- predict(mex.env, sp1_bioclim)

# Graficar la predicción del ENM
plot(sp1.map)

# Evaluar el modelo ENM
group <- kfold(sp1_points_nonas, 5)
pres_train <- sp1_points_nonas[group != 1, ]
pres_test <- sp1_points_nonas[group == 1, ]

backg <- randomPoints(mex.env, n=1000, ext=mex.extent, extf = 1.25) 
colnames(backg) = c('lon', 'lat')
group <- kfold(backg, 5)
backg_train <- backg[group != 1, ]
backg_test <- backg[group == 1, ]

e <- evaluate(pres_test, backg_test, sp1_bioclim, mex.env)

# Establecer un "threshold" para cortar la predicción y generar un mapa binario
threshold <- e@t[which.max(e@TPR + e@TNR)]
plot(sp1.map > threshold, main='presence/absence')

# Agregar los puntos "reales" de la especie 
points(sp1_points_mat[,1],sp1_points_mat[,2],cex=0.2)
```

¿Cómo se ve?

¿Hay regiones en donde la especie fue "predicha" (en realidad, las condiciones) pero en las que sabemos que no está?

**NOTA:** Salven el "workspace" de R, es decir, todo lo que se hizo en este ejercicio debe quedar salvado como un archivo ".Rdata". Estos objetos serán usados en ejercicios posteriores. NO CIERREN R SIN HABER GUARDADO ESTE ARCHIVO!!!
