---
title: "Construcion de mapas"
author: "Daniel Valencia, Juliana Herrera, Fabricio Villalobos"
date: "25-09-2025"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
```

## Paquetes necesarios

-   📦 **readr**: lectura/escritura rápida de archivos
-   📦 **dplyr**: manipulación de datos
-   📦 **sf**: manipulacion vectorial
-   📦 **terra**: manejo de raster y vectores
-   📦 **alphahull**: construcción de EOO a partir de puntos
-   📦 **rnaturalearth**: datasets geográficos (países) en `sf`
-   📦 **rnaturalearthdata**: datos complementarios para `rnaturalearth`
-   📦 **ggplot2**: visualización de gráficos
-   📦 **patchwork**: ordenar varios gráficos de `ggplot2`
-   📦 **scales**: escalas y etiquetas para `ggplot2`
-   📦 **scico**: paletas de color claras y consistentes

Cargamos los paquetes

```{r libraries, echo=TRUE, message=FALSE, warning=FALSE}
library(readr)
library(dplyr)
library(sf)
library(terra)
library(alphahull)
library(rnaturalearth)
library(rnaturalearthdata)
library(ggplot2)
library(patchwork)
library(scales)
library(scico)
source("../datos/clean_dup.R") # función para limpiar duplicados
```

# Procesos espaciales

En este ejercicio, vamos a utilizar un pez dulceacuícola (*Cyphocharax naegelii*), que se distribuye en la cuenca alta del río Paraná

Por ejemplo, para especies restringidas o dependientes de aguas continentales podemos implementar algunos ajustes luego de determinar su extensión de presencia

![](images/America_Parana.png)

------------------------------------------------------------------------

Entonces carguemos una base de datos con los registros de la especie ya limpios para la cuenca del alto Paraná

```{r}
datos <- read_csv("../datos/polygons/C_naegelii.csv") |>
  dplyr::select("longitude", "latitude") |> 
  mutate(longitude=ifelse(duplicated(longitude),longitude+rnorm(1,mean=0, sd=0.0001),longitude)) |> 
  mutate(latitude=ifelse(duplicated(latitude),latitude+rnorm(1,mean=0, sd=0.0001),latitude))
```

Y carguemos un shape de referencia para la cuenca del alto Paraná; disponibles en la base de datos [HydroSHEDS](https://www.hydrosheds.org/)

```{r}
M <- sf::st_read("../datos/polygons/Cyphocharax_naegelii.shp") |> 
  sf::st_transform(4326) |> sf::st_make_valid()
```

## Grafiquemos los registros y la cuenca

-   NOTA: Debemos convertir los registros del objeto **`datos`** a un objeto de tipo `sf` para poder graficar en `ggplot`

Veamos los registros distribuidos en la cuenca

```{r}
datos_sf <- sf::st_as_sf(datos, coords=c("longitude", "latitude"), crs=4326)

ggplot()+
  geom_sf(data=M, fill="white", linewidth= 0.5, colour="black")+
  geom_sf(data=datos_sf, colour="blue", size=1)+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)+
  labs(x="Longitud decimal", y="Latitud decimal",
       title=expression(paste("Registros reportados ", italic("Cyphocharax naegelli"))))+
  theme(plot.title=element_text(hjust=0.5))
```

Como vamos a hacer procesos espaciales como cortes o intersecciones espaciales, necesitamos algunos insumos complementarios:

1.  Otros shapes de cuencas, pero de menor orden (*i.e.*, cuencas más pequeñas); en este caso, subcuencas nivel 8 (HydroSHEDS)

```{r}
cuencas <- sf::st_read("../datos/polygons/Basins_8.shp") |> sf::st_transform(4326) |> sf::st_make_valid()
```

Grafiquemos las subcuencas para tener una referencia visual

```{r}
ggplot()+
  geom_sf(data=cuencas, fill="white", linewidth=0.1, colour="black")+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)
```

2.  Ráster de referencia que represente los cuerpos de agua (*e.g.*, ríos, embalse)

Dado que solo necesitamos el ráster como una referencia, asignaremos un valor único para todos los píxeles diferentes de NA

```{r}
mask <- terra::rast("../datos/polygons/rivers.tif")
mask[!is.na(mask)] <- 1
```

NOTA: Para graficar el ráster, primero hay que convertirlo en un **data.frame**

```{r}
mask.2 <- as.data.frame(mask, xy=TRUE)

ggplot()+
  geom_sf(data=M, fill="white", linewidth=0.5, colour="black")+
  geom_raster(data = mask.2, aes(x=x, y=y), fill="red")+
  coord_sf(crs=st_crs(4326),xlim=c(-58.2, -43.9),ylim=c(-28,-15),expand=TRUE)
```

Teniendo los registros, capas de cuencas, y ráster de ríos, calculemos la extensión de presencia de la especie y luego ajustémoslo a las subcuencas, y a los cuerpos de agua

-   Trazar el polígono/distribución de la espceie

-   Interceptar el polígono trazado con la capa de subcuencas (nivel 8; objeto **cuencas**)

-   Rasterizar el polígono interceptado y cortarlo a los cuerpos de agua utilizando la capa ráster de los ríos del objeto **mask**

## Construir el polígono de forma **alfa**

NOTA: Para observar el alpha hull, necesitamos que el objeto sea de tipo espacial del paquete `sf`. Para eso usaremos una función independiente (**ah2sf.R**), disponible en su carpeta de trabajo

```{r}
source(file="../datos/ah2sf.R")

naegelli_poly <- alphahull::ahull(datos, alpha=1.5) |> ah2sf()
```

¿Cómo se ve?

```{r}
ggplot()+
  geom_sf(data=M, fill="white", linewidth=0.5, colour="black")+
  geom_sf(data=naegelli_poly,fill="green",linewidth=0.2,colour="black",alpha=0.3)+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)
```

Interceptar el polígono con las subcuencas

```{r echo=TRUE, message=FALSE, warning=FALSE}
intersect_poly <- sf::st_intersection(cuencas, naegelli_poly)

ggplot()+
  geom_sf(data=cuencas, fill="white", linewidth= 0.1, colour="black")+
  geom_sf(data=intersect_poly$geometry,fill="green",linewidth=0.2,colour="black",alpha=0.3)+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)
```

Seleccionar del objeto **cuencas** los ID interceptados para unir las subcuencas seleccionadas

```{r echo=TRUE, message=FALSE, warning=FALSE}
range_union <- cuencas[(intersect_poly),] |> sf::st_union()

ggplot()+
  geom_sf(data=M, fill="white", linewidth= 0.5, colour="black")+
  geom_sf(data=range_union,fill="green",linewidth=0.2,colour="black",alpha=0.3)+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)
```

Por último, utilizamos el polígono ajustado a las subcuencas (objeto **range_union**) y lo rasterizamos. Luego, lo cortamos a los ríos usando el ráster del objeto **mask**

NOTA: Dado que el objeto final es una capa ráster, necesitamos transformarla en un data.frame para poder graficarla en `ggplot`

```{r echo=TRUE, message=FALSE, warning=FALSE}
water_restric <- terra::rasterize(sf::st_as_sf(terra::vect(range_union)), mask, getCover=TRUE) |> 
  terra::crop(mask, mask=TRUE, snap="near") |>
  as.data.frame(xy=TRUE)
```

Veamos la extensión de presencia ajustada a los ríos

```{r}
ggplot()+
  geom_sf(data=M, fill="white", linewidth=0.5, colour="black")+
  geom_raster(data=water_restric, aes(x=x, y=y), fill="red")+
  coord_sf(crs=st_crs(4326),xlim=c(-58.2, -43.9),ylim=c(-28,-15),expand=TRUE)
```

Grafiquemos ambos resultados para tener una aproximación visual

```{r}
unrestrict_range <- ggplot()+
  geom_sf(data=M, fill="white", linewidth= 0.5, colour="black")+
  geom_sf(data=range_union,fill="green",linewidth=0.2,colour="black",alpha=0.3)+
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)+
  labs(x="Longitud decimal",y="Latitud decimal",
       title=expression(paste("Sin restricciones a cuerpos de agua")))+
  theme(plot.title=element_text(hjust=0.5))

restrict_range <- ggplot()+
  geom_sf(data=M, fill="white", linewidth=0.5, colour="black")+
  geom_raster(data = water_restric, aes(x=x, y=y), fill="red") +
  coord_sf(crs=st_crs(4326), xlim=c(-58.2, -43.9), ylim=c(-28,-15), expand=TRUE)+
  labs(x="Longitud decimal",y="",title=expression(paste("Restringido a cuerpos de agua")))+
  theme(plot.title=element_text(hjust=0.5))

(unrestrict_range | restrict_range) +
  plot_layout(widths = c(1, 1))
```

En resumen, vemos todo el proceso espacial que realizamos en esta primera parte para tener una visión general

![](images/Poligonos.png)

------------------------------------------------------------------------

# Extracción datos climaticos

## Para este ejemplo usaremos:

-   La base de datos de *Bothrops asper* que descargamos de **GBIF**

```{r echo=TRUE, message=FALSE, warning=FALSE}
asper <- read_csv("../datos/B_asper.csv") |> 
  dplyr::filter(basisOfRecord != "FOSSIL_SPECIMEN") |>
  dplyr::select(species, longitude=decimalLongitude, latitude=decimalLatitude) |>
  clean_dup(longitude="longitude", latitude="latitude", threshold=0.04166667) |>
  tidyr::drop_na() |> 
  dplyr::distinct()

print(asper)
```

-   Variable de elevación obtenida de [WorldClim 2.1](https://www.worldclim.org/data/worldclim21.html)

```{r}
elev <- terra::rast("../datos/wc2.1_2.5m_elev_am.tif")

print(elev)
```

Convertir el ráster de elevación en un **data.frame** para gráficar

```{r echo=TRUE, message=FALSE, warning=FALSE}
elev_df <- as.data.frame(elev, xy=TRUE)

ggplot(elev_df, aes(x=x, y=y, fill=wc2.1_2.5m_elev)) +
  geom_raster() +
  scale_fill_viridis_c() +
  coord_equal()
```

Ahora si, extraigamos los datos de elevacón asociados a los puntos de presencia de *B. asper*

```{r echo=TRUE, message=FALSE, warning=FALSE}
# Convertir los puntos a SpatVector (lon/lat)
pts <- terra::vect(asper, geom=c("longitude","latitude"), crs=4326)

# Extraer elevación
elev_vals <- terra::extract(elev, pts, ID=FALSE, method="bilinear")

# Renombrar y unir a la tabla de presencias
names(elev_vals) <- "elev_m"
asper_elev <- dplyr::bind_cols(asper, elev_vals) |> tidyr::drop_na(elev_m)

print(asper_elev)
```

¿Dónde se concentran mis registros por latitud?

```{r echo=TRUE, message=FALSE, warning=FALSE}
den <- ggplot(asper_elev, aes(x=latitude)) +
  geom_density() + theme_bw() + labs(x="Latitud", y="Densidad")

# Fondo de países para mapear 
americas <- rnaturalearth::ne_countries(scale="medium", returnclass="sf") |>
  dplyr::filter(admin %in% c("Mexico","Guatemala","Belize","Honduras","El Salvador",
                             "Nicaragua","Costa Rica","Panama","Colombia","Ecuador"))

# Curva de densidad sobre latitud
d <- density(asper_elev$latitude)
asper_elev$lat_dens <- approx(d$x, d$y, xout=asper_elev$latitude)$y

den_map <- ggplot() +
  geom_sf(data=americas, fill="grey90", color="white", linewidth=0.2) +
  geom_point(data=asper_elev, aes(longitude, latitude, color=lat_dens), size=0.7, alpha=0.8) +
  scale_color_viridis_c(name="Densidad", guide=guide_colorbar(barwidth=0.5, barheight=4)) +
  labs(x="Longitud", y="Latitud") +
  theme_minimal() + 
  theme(legend.position=c(0.15, 0.3), 
        legend.title=element_text(size=8, hjust=0.5),
        legend.text=element_text(size=8))

den + den_map
```

¿Cómo se distribuye la elevación de mis presencias?

```{r echo=TRUE, message=FALSE, warning=FALSE}
dis_elev <- ggplot(asper_elev, aes(x=elev_m))+geom_density()+theme_bw()+
  labs(x="Elevación", y="Densidad", title="Distribución de presencias")

# Curva de densidad sobre elevación
dele <- density(asper_elev$elev_m, na.rm=TRUE)
asper_elev$elev_dens <- approx(dele$x, dele$y, xout=asper_elev$elev_m)$y

map_elev <- ggplot() +
  geom_raster(data=elev_df, aes(x=x, y=y, fill=wc2.1_2.5m_elev)) +
  scale_fill_gradient(low="grey90", high="grey60", guide="none") +
  geom_point(data=asper_elev,aes(x=longitude,y=latitude,color=elev_dens), size=0.7) +
  scale_color_viridis_c(name="Densidad\n(elevación)",labels=scales::label_number(accuracy=0.001),
                        guide=guide_colorbar(barwidth=0.5, barheight=4)) +
  coord_sf(xlim=c(-110, -70), ylim=c(-10, 25), expand=FALSE) +
  theme_minimal()+
  theme(legend.position=c(0.15, 0.3), 
        legend.title=element_text(size=8), 
        legend.text=element_text(size=8))

dis_elev + map_elev
```

# Midpoints

Para este ejemplo usaremos un shapefile multipolígono de mapas de distribución de mamíferos descargado de la página de [Recursos de Datos Espaciales y Cartografía de la Lista Roja](https://www.iucnredlist.org/resources/spatial-data-download) de la **UICN** (descarga gratuita y abierta para uso educativo, pero se requiere una cuenta)

```{r  echo=TRUE, message=FALSE, warning=FALSE}
mammalpolys <- sf::st_read("../datos/polygons/MAMMALS_TERRESTRIAL_ONLY.shp")

# Generemos un mapa base de Ameria como área de estudio
worldmap <- rnaturalearth::ne_countries(scale="small", returnclass="sf")
americas <- worldmap |> dplyr::filter(region_un == "Americas") |> sf::st_make_valid()
```

------------------------------------------------------------------------

> Como partimos de todos los mamíferos del mundo, filtramos solo los **PRIMATES**. Además, muchos *shapefiles* globales traen polígonos inválidos (huecos anómalos, autointersecciones), por lo que `sf` puede rechazarlos al intersectar o filtrar. Con `st_make_valid` reparamos la geometría antes de utilizarlos

```{r}
primate_polys <- mammalpolys |> dplyr::filter(order_ == "PRIMATES") |> 
  sf::st_make_valid()
```

Ahora implementemos algunas operaciones como recortar, unir y combinar

```{r echo=TRUE, message=FALSE, warning=FALSE}
# Desactivar s2 solo para recortar
# Algunos polígonos vienen con fallas, deshabilitar s2 evita que al cortar se generen errores 
old_s2 <- sf::sf_use_s2()
sf::sf_use_s2(FALSE)

# Recortar por el rectángulo de Américas, conservando cualquier geometría que toque el rectángulo
primates_bb <- sf::st_crop(primate_polys, sf::st_bbox(americas))

# Unir todos los países de Américas en una máscara y recorta los rangos al contorno continental
americas_poly <- sf::st_union(americas) |> sf::st_make_valid()
primates_ame  <- sf::st_intersection(primates_bb, americas_poly) |> sf::st_make_valid()

# Restaurar s2 para seguir con las operaciones en WGS84
sf::sf_use_s2(old_s2)

# Combinar los polígonos con el mismo sci_name en una sola geometría (i.e., una fila por especie)
primates_ame <- primates_ame |> group_by(sci_name) |> summarize()
```

¿Cómo se ven las distribuciones/EOO?

Seleccionemos 12 especies y grafiquemos en facetas para evitar sobreposición

```{r}
primates_ame12 <- primates_ame |> dplyr::slice_sample(n=12)
lims_ame <- sf::st_bbox(primates_ame12)
divpol   <- americas |> dplyr::select(admin)

ggplot() +
  geom_sf(data=primates_ame12,aes(fill=sci_name),alpha=0.5,color="black",size=0.4,show.legend=FALSE) +
  facet_wrap(~sci_name) +
  geom_sf(data=divpol, color="gray40", fill=NA, size=0.2, alpha=0.5) +
  coord_sf(xlim=c(lims_ame["xmin"], lims_ame["xmax"]),
           ylim=c(lims_ame["ymin"], lims_ame["ymax"])) +
  scale_fill_scico_d(palette="hawaii") +
  theme_minimal() +
  theme(strip.text=element_text(face="italic"))
```

## ¿Qué podemos explorar con estos rangos?

-   Por ejemplo, patrones espaciales y/o tamaño de las distribuciones

Aseguremonos de trabajar en WGS84 (lat/lon). Así podemos calcular los midpoints y la extensión latitudinal de los rangos

```{r echo=TRUE, message=FALSE, warning=FALSE}
pc_ll <- primates_ame |> sf::st_make_valid() |> dplyr::group_by(sci_name) |>
  summarize(.groups="drop") |> sf::st_transform(4326)
```

Calculemos el midpoint de cada especie (es decir, centroide dentro del polígono)

```{r echo=TRUE, message=FALSE, warning=FALSE}
cent <- sf::st_point_on_surface(pc_ll) |> sf::st_coordinates()

pc_mid <- pc_ll |> dplyr::mutate(mid_lon=cent[,1], mid_lat=cent[,2])
print(pc_mid)
```

Extraemos por especie las latitudes mínima y máxima, y agregamos toda esta informacion en el `data.frame`

```{r echo=TRUE, message=FALSE, warning=FALSE}
bbs  <- lapply(st_geometry(pc_ll), st_bbox)
ymin <- sapply(bbs, function(b) b["ymin"])
ymax <- sapply(bbs, function(b) b["ymax"])

df <- pc_mid |> mutate(ymin=ymin,ymax=ymax,lat_range=pmax(0, ymax-ymin),abs_mid_lat=abs(mid_lat)) |>
  sf::st_drop_geometry() |> dplyr::filter(lat_range > 0)

print(df)
```

Muchos cálculos y datos en tabla... ¿y esto qué nos dice?

pongámoslos en un mapa a ver qué pasa en la geografía

```{r}
amer_ll <- rnaturalearth::ne_countries(scale="small", returnclass="sf") |> 
  dplyr::filter(region_un=="Americas") |> sf::st_transform(4326)

ggplot() +
  geom_sf(data=amer_ll, fill="grey95", color="grey50", size=0.3) +
  geom_sf(data=pc_ll, fill=NA, color="grey70", size=0.2) +
  geom_hline(yintercept=0, linetype="dashed", color="grey50") +
  geom_point(data=df, aes(x=mid_lon, y=mid_lat, color=lat_range), size=1, alpha=0.9) +
  scale_color_viridis_c(name="Extensión latitudinal del rango (° N-S)") +
  coord_sf(xlim=st_bbox(amer_ll)[c("xmin","xmax")], ylim=st_bbox(amer_ll)[c("ymin","ymax")]) +
  labs(title="Midpoints de rangos geográficos (Primates, Américas)", x="Longitud", y="Latitud") +
  theme_minimal()
```

> NOTA: El color **NO** depende de dónde esté el punto. Un punto amarillo (valor alto) cerca del ecuador indica una especie con una extensión amplia (p. ej., de 20°S a 20°N)

------------------------------------------------------------------------

Pero si queremos ver los midpoints en términos de las especies con mayor área de distribución.

Podemos reproyectar los rangos.

Para esto, usemos un **CRS** de área equivalente (Mollweide) para que `st_area` sea interpretable

```{r echo=TRUE, message=FALSE, warning=FALSE}
crs_equal_area <- "+proj=moll +lon_0=0 +datum=WGS84 +units=m +no_defs"

pc_moll <- sf::st_transform(pc_ll, crs_equal_area)
area_km2 <- as.numeric(sf::st_area(pc_moll)) / 1e6 #para pasar de m² a km²

# Agregar el área al data.frame que ya contiene midpoint y extensión latitudinal
df_area <- df |> dplyr::mutate(area_km2=area_km2)
print(df_area)
```

Y volvemos a ver en la geografía. En este caso, el mapa se colorea por **área**

```{r echo=TRUE, message=FALSE, warning=FALSE}
ggplot() +
  geom_sf(data=amer_ll, fill="grey95", color="grey50", size=0.3) +
  geom_sf(data=pc_ll, fill=NA, color="grey70", size=0.2) +
  geom_hline(yintercept=0, linetype="dashed", color="grey50") +
  geom_point(data=df_area, aes(x=mid_lon, y=mid_lat, color=area_km2), size=0.8, alpha=0.9) +
  scale_color_viridis_c(name="Área del rango (km²)") +
  coord_sf(xlim=st_bbox(amer_ll)[c("xmin","xmax")], ylim=st_bbox(amer_ll)[c("ymin","ymax")]) +
  labs(title="Midpoints y área del rango (Primates, Américas)", x="Longitud", y="Latitud") +
  theme_minimal()
```
