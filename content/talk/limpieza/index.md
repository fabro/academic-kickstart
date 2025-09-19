---
title: "Datos geográfico: limpieza"
author: "Daniel Valencia, Juliana Herrera, Fabricio Villalobos"
date: "25-09-2025"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
```

> **Objetivo.** Introducir un flujo de trabajo mínimo para limpiar datos de presencia de especies provenientes de diversas fuentes

## Paquetes para hoy

-   📦 **readr**: lectura/escritura rápida de archivos
-   📦 **dplyr**: manipulación de datos
-   📦 **sf**: manipulacion vectorial
-   📦 **rnaturalearth**: datasets geográficos (países) en **sf**
-   📦 **mapview**: mapas interactivos para objetos **sf**/**raster**
-   📦 **ggplot2**: visualización de gráficos
-   📦 **patchwork**: ordenar varios gráficos de ggplot2

Cargamos los paquetes

```{r libraries, echo=TRUE, message=FALSE, warning=FALSE}
library(readr)
library(dplyr)
library(sf)
library(rnaturalearth)
library(mapview)
library(ggplot2)
library(patchwork)
```

Carguemos la base de datos con los registros obtenidos del **GBIF**

```{r}
registros <- read_csv("../datos/B_asper.csv")
names(registros)
```

Miremos los datos en la geografía para tener un panorama general de su distribución espacial

```{r}
# espacialicemos los datos
occs_geo <- 
  st_as_sf(registros, coords=c("decimalLongitude", "decimalLatitude"), crs="EPSG:4326")

# Agregar un mapa de las Americas para saber qué onda!
americas <- ne_countries(scale="small", continent=c("North America", "South America"),
  returnclass="sf")

# y grafiquemos
ggplot()+ 
  geom_sf(data=americas)+
  geom_sf(data=occs_geo, col="blue", pch=19, size=0.3)+
  coord_sf(expand=FALSE) +
  labs(x="Longitud decimal", y="Latitud decimal",
       title=expression(paste("Puntos reportados ", italic("Bothrops asper"))))+
  theme(plot.title=element_text(hjust=0.5))
```

Exploremos un poco los datos

```{r}
ggplot(registros, aes(x=basisOfRecord, fill=NULL))+ 
  geom_bar() + theme_bw()+ 
  labs(x="Tipo de registro", "Número de registros")+
  theme(legend.position="none") +

ggplot(registros, aes(x=country, fill=NULL))+ 
  geom_bar() + theme_bw() + 
  labs(x="Pais", "Número de registros")+
  theme(legend.position="none") +

ggplot(registros, aes(x=decimalLatitude))+ 
  geom_density() + theme_bw() + 
  labs(x="Latitud", "Densidad") +

plot_layout(ncol=1)
```

Ya tenemos una idea "muy" general de nuestros datos; pero estos datos están tal cual los descargamos de **GBIF**

## Curaduría espacial

> Con esto nos referimos a la aplicación de algunos filtros simples para depurar datos que no son de nuestro interés; esto varía según la pregunta de investigación

## En este caso vamos a eliminar:

-   Registros fósiles
-   Registros sin georreferencia
-   Registros duplicados
-   Transformar los registros en un objeto espacial

```{r}
occ <- registros |> 
  dplyr::filter(basisOfRecord != "FOSSIL_SPECIMEN") |> #eliminar fósiles 
  dplyr::select(species, longitude=decimalLongitude, latitude=decimalLatitude) |> 
  tidyr::drop_na() |> #elimina NAs
  dplyr::distinct()   #elimina duplicados

occ
```

Aún tenemos muchos datos, no?

```{r}
# Espacialicemos los registros
occs_geo.2 <- st_as_sf(occ, coords=c("longitude", "latitude"), crs="EPSG:4326")

# y grafiquemos
ggplot()+ 
  geom_sf(data=americas)+
  geom_sf(data=occs_geo.2, col="blue", pch=19, size=0.3)+
  coord_sf(expand=FALSE) +
  labs(x="Longitud decimal", y="Latitud decimal",
       title=expression(paste("Puntos reportados ", italic("Bothrops asper"))))+
  theme(plot.title=element_text(hjust=0.5))
```

Pero... si por alguna razón solo quiero los registros alguna región (p.ej., Colombia)

Eliminar los puntos que están por fuera de Colombia

```{r}
occ.2 <- occ |>
  sf::st_as_sf(coords=c("longitude", "latitude"), crs=4326, remove=FALSE) |>
   sf::st_crop(xmin=-78, xmax=-75, ymin=0.5, ymax=11)

## otra alternativa de filtrado podría ser
#occ.2 <- occ |>
#  sf::st_as_sf(coords=c("longitude","latitude"), crs=4326, remove=FALSE) |>
#  dplyr::filter(between(latitude, 0.5, 11), between(longitude, -78, -75))
```

Mapeamos de nuevo, pero solamente en la región de interés (Colombia)

```{r}
col_map <- dplyr::filter(americas, name=="Colombia")

ggplot()+ 
  geom_sf(data=col_map)+
  geom_sf(data=occ.2, col="blue", pch=19, size=1)+
  coord_sf(expand=FALSE) +
  labs(x="Longitud decimal", y="Latitud decimal",
       title=expression(paste("Puntos reportados ", italic("Bothrops asper"))))+
  theme(plot.title=element_text(hjust=0.5))
```

Y ¿cómo eliminamos los registros que están en el "mar"?

```{r}
mapview::mapview(occ.2)
```

Por último, exportemos nuestra base de datos con los datos **"limpios"**

```{r}
readr::write_csv(occ.2, "../datos/B_asper_limpios.csv")
```
