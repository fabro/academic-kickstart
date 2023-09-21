+++
title = "Species geographic distributions"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Presencias y distribuciones geográficas

**Macroecología**

Instituto de Ecología, A.C.  - INECOL

##### parte del código está basado en la vignette del paquete `dismo` (Hijmans & Elith 2011), código de Vijay Barve (https://vijaybarve.wordpress.com) y Luis Verde-Arregoitia (https://luisdva.github.io/)

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
library(alphahull)
library(rangeBuilder)

```

Obtener datos de presencia de una especie cualquiera a partir de la base de datos GBIF (data.gbif.org) directamente desde R!
**NOTA**: Necesitan estar conectados a internet. Este proceso puede tardar, dependiendo de la especie, si esta tiene (o no) muchos registros

```{r eval=FALSE}
sp_datos <- occ_data(scientificName = "Musonycteris harrisoni", limit = 500)
```
**NOTA**: Uds podrían colocar el nombre del género y la especie deseada.

El objeto `sp_datos` es una lista con datos sobre los resultados obtenidos en GBIF, para trabjar únicamente con la tabla de registros hay que seleccionar el objeto _data_ dentro del mismo
```{r eval=FALSE}
sp_datos <- sp_datos$data
```

Ver las dimensiones del objeto generado (i.e. cuántas líneas y cuántas columnas tiene)
```{r eval=FALSE}
dim(sp_datos)
```
Checar el nombre de las columnas (para después buscar únicamente las de posición geográfica: lat/long)
```{r eval=FALSE}
names(sp_data)[1:30]
```

Crear otro objeto a partir del anterior para quedarse únicamente con long/lat. Quedarse únicamente con los puntos/registros individuales (i.e., excluir duplicados). Transformarlo en un objeto espacial

```{r eval=FALSE}
sp_p1<-sp_data %>%
	select(decimalLongitude,decimalLatitude,species) %>%
	mutate(lat=decimalLatitude,lon=decimalLongitude) %>%
	distinct() %>%
	na.omit() %>% 
	sf::st_as_sf(coords = c('decimalLongitude','decimalLatitude'),crs="EPSG: 4326")

```
**NOTA**: el nombre de la variable puede ser diferente (.e.g "LATITUDE", "Latidude", "lat", etc. Siempre hay que checar antes)

Graficar (poner en un mapa) los puntos de presencia de nuestra especie
```{r eval=FALSE}
ggplot()+ 
	geom_sf(data=sp_p1, col="blue",pch=19)
```

Agregar el mapa del mundo. Para eso, necesitamos cargar el mapa (que viene con el paquete "rnaturalearth")
```{r eval=FALSE}
wrld <- ne_countries(scale = "small",returnclass = "sf") 
```

```{r eval=FALSE}
ggplot()+
	geom_sf(data=wrld)+geom_sf(data=sp_p1,col="blue",pch=19,size=1)+coord_sf(expand = F) +
	labs(x="Longitud decimal ",
	y="Latitud decimal",
	title=expression(paste("Puntos reportados ", italic("Glossophaga morenoi"))))+
	theme(plot.title = element_text(hjust = 0.5))
```

¿Hay algún dato extraño? Los puntos/registros necesitan ser "curados" (limpiados)

Dependiendo de la especie, el mapa puede ser muy grande, ¿cierto? Da para generar un mapa "menor", sabiendo en dónde están nuestros puntos.
En ese caso, vamos a crear un mapa sólo para Argentina (o pra cualquer país, sólo hay que cambier el "NAME")

```{r eval=FALSE}
mex_map <- filter(wrld,name=="Mexico")

ggplot()+
	geom_sf(data=mex_map)+
	geom_sf(data=sp_p1,col="blue",pch=19,size=1)+
	coord_sf(expand = F)
```

# Ejercicio 2

## Range maps from point data
En esta parte, se usaran funciones para generar mapas "simples" basados en la geometría (e.g. distancias/ángulos entre puntos, etc.). NO consideran variables ambientales!

## Convex hull (minimum convex polygon)
Este modelo crea un polígono convexo alrededor de los puntos de presencia.
```{r eval=FALSE}
sp1_mcp <- st_convex_hull(st_union(sp_p1))
```

¿Cómo se ve? 

```{r eval=FALSE}`
ggplot()+
  geom_sf(data=mex_map)+
  geom_sf(data=sp1_mcp,
          fill="blue")
```		  

## Polígono alfa (alpha hull)

Usamos el paquete `alphahull`

NOTA: Esta función solo acepta tablas como entrada

```{r eval=FALSE}
sp_p2<-as.data.frame(st_coordinates(sp_p1))

sp1_alphahull <- ahull(sp_p2, alpha = 6)
```

Para observar el alpha hull, necesitamos que el objeto sea de tipo espacial del paquete `sf`. Para eso usaremos una función independiente, disponible en su carpeta de trabajo

```{r eval=FALSE}
source(file = "ah2sf.R")
sp1_alphahull.poly <- ah2sf(sp1_alphahull)
```


¿Cómo se ve?

```{r eval=FALSE}
ggplot()+
	geom_sf(data=mex_map)+geom_sf(data=sp1_alphahull.poly,fill="blue")
```

## Polígono alfa dinámico

Usamos el paquete `rangeBuilder`, el cual crea un polígono alpha hull con un valor de alpha "óptimo" basado en la distribución espacial de los puntos.

```{r eval=FALSE}
sp_range <- getDynamicAlphaHull(
  sp_p2, #Tabla de puntos/registros de la especie
  coordHeaders = c("decimalLongitude", "decimalLatitude"),# x y y
  fraction = 0.95,   # la fracción mínima de registros que debe incluir el polígono
  partCount = 2,  # el máximo de polígonos disyuntos permitidos
  initialAlpha = 1, # Alpha inicial
  alphaIncrement = 0.5,
  alphaCap = 1000,
  clipToCoast = "terrestrial"  # solo la parte terrestre del polígono se mantendrá (se cortan las partes no-terrestres/acuáticas con base en un mapa descargado de naturalearth).
)
```

```{r eval=FALSE}
alpha <- sp_range[[2]]
alpha
```

Convertir el polígono alpha a un objeto sf

```{r eval=FALSE}
sp1_dynalpha <- st_make_valid(st_as_sf(sp_range[[1]]))
```

¿Cómo podemos visualizarlo?

```{r eval=FALSE}
ggplot()+ geom_sf(data=mex_map)+ 
  geom_sf(data=sp1_dynalpha, fill="blue")

```

## Y ....¿Cómo se ven todos los polígonos?

```{r eval=FALSE}
ggplot()+
  geom_sf(data=mex_map)+ geom_sf(data=sp1_mcp,fill="red",alpha=0.1) +
  geom_sf(data=sp1_alphahull.poly,fill="yellow",alpha=0.5) +
  geom_sf(data=sp1_dynalpha, fill="blue",alpha=0.5)
```

Finalmente, podemos salvar esos polígonos como `shapefiles`, para usarlos en otros software (e.g. ArcGIS) y eventualmente juntar los de varias especies para otros análisis (ejercicio siguiente)

```{r eval=FALSE}
st_write(sp1_mcp, "sp1_min_convex.shp")
st_write(sp1_alphahull.poly, "sp1_alphahull.shp")
st_write(sp1_dynalpha, "sp1_dyn_alphahull.shp")
```


# Ejercicio 3

Ahora, vamos a incluir datos ambientales/climáticos

Bajar los datos de clima del site www.worldclim.org (esto puede tardarse un poco, un par de minutos)
```{r eval=FALSE}
world_bioclim<-getData("worldclim", var = "bio", res = 10)
```

```{r eval=FALSE, echo=FALSE, results='hide',message=FALSE}
world_bioclim <- getData(name='worldclim',res=10,var='bio',path="/Users/fabricio/Library/CloudStorage/Dropbox/Macroecologia_2023/",download = F)
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


# Ejercicio 5
##Ecological Niche Model (Bioclim)

Vamos a quedarnos únicamente con dos variables
```{r eval=FALSE}
bio1<- world_bioclim_inds[[1]]
bio12<- world_bioclim_inds[[12]]
env.variables<- stack (bio1, bio12)
```

Establecer el "extent" de América del Sur y cortar las variables para ese extent
```{r eval=FALSE}
am.extent <- c(-100,-30,-60,14)
am.env <- crop(env.variables,am.extent)
```

Generar el modelo Bioclim (Ecological Niche Model: ENM)
```{r eval=FALSE}
# Convertir los puntos de la especie en un objeto tipo `matrix` 
sp1_points_mat <- as.matrix(sp1_points_nonas)

# Correr el ENM (Bioclim)
sp1_bioclim <- bioclim(am.env, sp1_points_mat)
plot(sp1_bioclim, a=1, b=2, p=0.85)

# Generar una predicción del área adecuada ("suitable") basada en el ENM
sp1.map<- predict (am.env, sp1_bioclim)

# Plotar o resultado da predição baseada no ENM
plot (sp1.map)

# Evaluar el modelo ENM
group <- kfold(sp1_points_nonas, 5)
pres_train <- sp1_points_nonas[group != 1, ]
pres_test <- sp1_points_nonas[group == 1, ]

backg <- randomPoints(am.env, n=1000, ext=am.extent, extf = 1.25) 
colnames(backg) = c('lon', 'lat')
group <- kfold(backg, 5)
backg_train <- backg[group != 1, ]
backg_test <- backg[group == 1, ]

e <- evaluate(pres_test, backg_test, sp1_bioclim, am.env)

# Establecer un "threshold" para cortar la predicción y generar un mapa binario
threshold <- e@t[which.max(e@TPR + e@TNR)]
plot(sp1.map > threshold, main='presence/absence')

# Agregar los puntos "reales" de la especie 
points(sp1_points_mat[,1],sp1_points_mat[,2],cex=0.2)
```

¿Cómo se ve?

¿Hay regiones en donde la especie fue "predicha" (en realidad, las condiciones) pero en las que sabemos que no está?

**NOTA:** Salven el "workspace" de R, es decir, todo lo que se hizo en este ejercicio debe quedar salvado como un archivo ".Rdata". Estos objetos serán usados en ejercicios posteriores. NO CIERREN R SIN HABER GUARDADO ESTE ARCHIVO!!!
