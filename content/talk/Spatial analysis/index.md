+++
title = "Spatial Filters"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

---
# Filtros espaciales: agregando el "espacio" en la explicación de la riqueza

**Macroecología**

Posgrado en Ciencias

Instituto de Ecología, A.C.
---


Cargar los siguientes paquetes:
```{r eval=FALSE}
library(letsR)
library(sf)
library(terra)
library(ape)
library(vegan)
```

Usaremos los datos de las aves endémicas de México (Villalobos et al. 2013. Biological Conservation), pero a una resolución menor (1º) para facilitar los análisis. Las variables climáticas fueron generados a partir de los datos de Cuervo-Robayo A. P., O. Téllez-Valdés, M. Gómez, C. Venegas-Barrera, J. Manjarrez y E. Martínez-Meyer. (2013). An update of high-resolution monthly climate surfaces for Mexico. International Journal of Climatology. Doi: 10.1002/joc.3848.(http://idrisi.uaemex.mx/ligas/geogadatos?id=8)

Cargar los datos bióticos (riqueza de aves endémicas) y ambientales (clima de México: temperatura, precipitación y sus estacionalidades). 
```{r eval=FALSE}
# Datos de riqueza de aves endémicas
load("ejercicios_datos/aves_geo.RData")

# Datos de clima para México
load("ejercicios_datos/mex_clim.RData")
```

Cargar un shapefile de México, sólo para referencia
```{r eval=FALSE}
mexico <- st_read("ejercicios_datos/destdv1gw/destdv1gw.shp")
```

Primero, generamos un raster con las características deseadas
```{r eval=FALSE}
mex.ras <- rast()
# con la misma extensión que los datos de las aves (México)
ext(mex.ras) <- ext(aves.geo)
# resolución de 1º
res(mex.ras) <- 1
ncell.mexras <- ncell(mex.ras)
# obtener las coordenadas de las celdas del raster
mexras.coords <- xyFromCell(object = mex.ras, cell = 1:ncell.mexras)
names(mexras.coords) <- c("Longitude(x)", "Latitude(y)")
```

```{r eval=FALSE}
# temperatura promedio anual
mex.temp <- rast("wc10/bio1.bil")
# estacionalidad de temperatura
mex.temp.sz <- rast("wc10/bio4.bil")
# precipitación total anual
mex.pp <- rast("wc10/bio12.bil")
# estacionalidad de precipitación
mex.pp.sz <- rast("wc10/bio15.bil")
```

Ahora podemos rasterizar el mapa de riqueza original y las variables ambientales para resolución de 1º
```{r eval=FALSE}
aves.geo <- st_as_sf(aves.geo)
aves.geo.ras <- rasterize(aves.geo, mex.ras, field = "spp_rich", fun = mean)
# Cómo se ve?
mexico <- as(mexico, "Spatial") # de sf a sp
plot(aves.geo.ras)
plot(mexico, add = T)
# rasterizar los valores de las variables climáticas a 1º (originalmente a 1km2)
mex.temp1 <- extract(mex.temp, mexras.coords)
mex.temp.sz1 <- extract(mex.temp.sz, mexras.coords)
mex.pp1 <- extract(mex.pp, mexras.coords)
mex.pp.sz1 <- extract(mex.pp.sz, mexras.coords)
```

Crear un solo conjunto de datos
```{r eval=FALSE}
aves.ras1.data <- as.data.frame(cbind(mexras.coords, values(aves.geo.ras), mex.pp1, mex.pp.sz1, mex.temp1, mex.temp.sz1))
names(aves.ras1.data) <- c("longitude", "latitude", "spp_richness", "total_precipitation", "precipitation_seazonality", "mean_annual_temperature", "temperature_seazonality")
# Checar que no haya NAs, si los hay, eliminarlos
aves.ras1.data.comp <- na.omit(aves.ras1.data)
```

Ajustar el modelo global (OLS) entre la riqueza de aves y las variables ambientales
```{r eval=FALSE}
aves.ras1.lm <- lm(spp_richness ~ mean_annual_temperature + temperature_seazonality + total_precipitation + precipitation_seazonality, data = aves.ras1.data.comp)
summary(aves.ras1.lm)
```

Checar la autocorrelación espacial en los residuos del modelo
```{r eval=FALSE}
# Necesitamos crear una matriz de distancias
aves.ras1.coords.dist <- lets.distmat(aves.ras1.data.comp[, c(1, 2)])
# Ahora sí podemos calcular el I de Moran para diferentes clases de distancia y ver el correlograma resultante
aves.ras1.lm.moran <- lets.correl(residuals(aves.ras1.lm), aves.ras1.coords.dist, 10)
```
¿Qué tal? ¿Existe autocorrelación? ¿Mucha/Poca? ¿En cuántas clases de distancia?

## Eigenvector-based Spatial Filters

Paso 1. Crear la matriz de distancias, "truncarla" usando la distancia identificada en el autocorrelagrama de Moran (cuál sería?) e invertirla para que las distancias cortas tengan más peso

```{r eval=FALSE}
# Crear la matriz de distancia, como un objeto "matrix" en lugar de "dist"
mat.dist <- lets.distmat(aves.ras1.data.comp[, c(1, 2)], asdist = FALSE)

# Generar la matriz truncada, con base en la distancia en donde ya no tenemos autocorrelación (checar con el correlagrama de Moran creado arriba)
mat.dist.w <- ifelse(test = mat.dist > 239, yes = 4 * 239, no = mat.dist)

# "Invertir" la matriz para que las distancias cortas tengan más peso: matriz de pesos "w".
# NOTA: este paso no es una "inversión" estadísticamente hablando de la matriz, sino simplemente una transformación de nuestros valores para que los más pequeños pesen más y los más altos pesen menos. Para invertir una matriz de manera estadística, vean la función "solve"
w <- 1 / mat.dist.w
diag(w) <- 0

# Checar el coeficiente de autocorrelación global (Moran's I) usando la nueva matriz de pesos "w"
Moran.I(residuals(aves.ras1.lm), w)
```

Paso 2. Conducir el "eigenanalysis" de la matriz de distancias (PCoA: análisis de coordenadas principales)

```{r eval=FALSE}
# PCoA - del paquete "ape". (La matriz tiene que ser cuadrada)
# NOTA: la función "pcoa" centraliza la matriz, por tanto no es necesario hacerlo antes
# NOTA2: el PCoA se construye a partir de la matriz truncada, no de la matriz de pesos.
aves.pcord <- pcoa(mat.dist.w, correction = "none", rn = NULL)

# Crear un objeto que "salve" los eigenvectors generados
aves.vecold <- aves.pcord$vectors
# cuántos vectores? salvar ese numero
n <- ncol(aves.vecold)
```

Paso 3. Selección de eigenvectores: secuencialmente con base en la minimización del índice I de Moran global (checado arriba)

```{r eval=FALSE}
# Crear un "loop" para ir seleccionado los vectores con base en el análisis de la autocorrelación espacial de los residuos del modelo global (ambiente + vectores)

for (i in 1:n - 1) {
  m1 <- lm(aves.ras1.data.comp$spp_richness ~ aves.vecold[, 1:i] + aves.ras1.data.comp$total_precipitation + aves.ras1.data.comp$precipitation_seazonality + aves.ras1.data.comp$mean_annual_temperature + aves.ras1.data.comp$temperature_seazonality)
  res <- m1$residuals
  mor <- Moran.I(res, w)
  p.mor <- mor$p.value
  if (p.mor > 0.05) {
    vec.sel <- (i + 2)
  }
  if (p.mor > 0.05) {
    break
  }
}

# salvar los vectores seleccionados
aves.vecsel <- aves.vecold[, 1:vec.sel]
```
¿Cuántos vectores fueron seleccionados?

## SEVM: spatial eigenvector mapping
¿Y si lo vemos en el mapa? 
```{r eval=FALSE}
# Rasterizar los vectores seleccionados, algunos como ejemplo (el primero y el último; amplia y pequeña escala, respectivamente)

# primer vector (amplia escala)
aves.vecsel1 <- rasterize(aves.ras1.data.comp[, c(1, 2)], raster(mex.ras), aves.vecsel[, 1])
plot(aves.vecsel1)
plot(mexico, add = T)

# último vector (pequeña escala)
aves.vecsel15 <- rasterize(aves.ras1.data.comp[, c(1, 2)], raster(mex.ras), aves.vecsel[, 15])
plot(aves.vecsel15)
```

¿Cómo se "ven"?


Paso 4. Ajustar un modelo incluyendo los vectores espaciales junto con los variables ambientales

```{r eval=FALSE}
aves.ras1.lm.vecs <- lm(aves.ras1.data.comp$spp_richness ~ aves.ras1.data.comp$total_precipitation + aves.ras1.data.comp$precipitation_seazonality + aves.ras1.data.comp$mean_annual_temperature + aves.ras1.data.comp$temperature_seazonality + aves.vecsel[, 1:15])

summary(aves.ras1.lm.vecs)
```
¿Qué tal, explicó más que el modelo anterior?


Paso 5. Checar la autocorrelación en los residuos del nuevo modelo

```{r eval=FALSE}
aves.ras1.lm.vecs.moran <- lets.correl(residuals(aves.ras1.lm.vecs), aves.ras1.coords.dist, 10)
```
¿Aún existe autocorrelación espacial? Si/No?


Paso 6. ¿Qué tanta variación en la riqueza de aves endémicas es explicada por el ambiente y por el espacio? (partición de varianza)

```{r eval=FALSE}
aves.varpart <- varpart(Y = aves.ras1.data.comp$spp_richness, aves.ras1.data.comp[, c(4:7)], aves.vecsel)

plot(aves.varpart)
```
¿Quién explicó más individualmente? ¿y en conjunto?

¿Qué podemos concluir a partir de estos resultados?