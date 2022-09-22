+++
title = "Spatial Filters"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Agregando el "espacio" al análisis de la riqueza

**Macroecología**

Universidad de Buenos Aires

Cargar los siguientes paquetes:
```{r eval=FALSE}
library(letsR)
library(sf)
library(terra)
library(maptools)
library(ape)
library(vegan)
```

Usaremos los datos de los carnívoros y de clima (WorldClim) que "generamos" en el ejercicio de gradientes y escalas (ejercicio 2)

Cargar los datos bióticos (riqueza de Carnivora en América del Sur) y ambientales. 
```{r eval=FALSE}
#Raster de riqueza (para no demorar mucho, vamos usar la escala/resolución de 2º)
carniv.rich <- carniv_raster2

#vamos a ver los datos, sólo para corroborar
plot(carniv.rich)
data("wrld_simpl")
plot(wrld_simpl,add=T)
```

Cargar los datos de clima global que ya tenemos.
```{r eval=FALSE}
world_bioclim <- getData(name='worldclim',res=10,var='bio',path="/Users/fabro/laboratorio Macroecología/Cursos/posgrado_INECOL/Macroecologia/",download = F)
#separar esos datos (layers)
world_bioclim_inds<-unstack(world_bioclim)
```

Obtener las coordenadas de nuestro raster
```{r eval=FALSE}
ncell.carniv <- ncell(carniv.rich)
carniv.coords <- xyFromCell(object = carniv.rich, cell = 1:ncell.carniv)
colnames(carniv.coords) <- c("Longitude(x)", "Latitude(y)")

```

Ahora, podemos usar las coordenadas del raster para extraer los datos ambientales en los lugares en donde tenemos carnívoros
```{r eval=FALSE}
carniv.temp2 <- extract(world_bioclim_inds[[1]], carniv.coords)
carniv.temp.sz2 <- extract(world_bioclim_inds[[4]], carniv.coords)
carniv.pp2 <- extract(world_bioclim_inds[[12]], carniv.coords)
carniv.pp.sz2 <- extract(world_bioclim_inds[[15]], carniv.coords)
```

Crear un solo conjunto de datos
```{r eval=FALSE}
carniv.ras2.data <- as.data.frame(cbind(carniv.coords,values(carniv.rich),carniv.pp2,carniv.pp.sz2,carniv.temp2,carniv.temp.sz2))
colnames(carniv.ras2.data) <- c("longitude","latitude","spp_richness","total_precipitation","precipitation_seazonality","mean_annual_temperature","temperature_seazonality")
#Checar que no tengan NAs. Si los hay, tenemos que eliminarlos
carniv.ras2.data.comp <- na.omit(carniv.ras2.data)
```

Ajustar el modelo global (OLS) entre la riqueza de carnívoros y las variables ambientales
```{r eval=FALSE}
carniv.ras2.lm <- lm(spp_richness ~ mean_annual_temperature + temperature_seazonality + total_precipitation + precipitation_seazonality, data=carniv.ras2.data.comp)
summary(carniv.ras2.lm)
```

Checar la autocorrelación espacial en los residuos del modelo
```{r eval=FALSE}
#Necesitamos crear una matriz de distancias geográficas entre los sitios/celdas
carniv.ras2.coords.dist <- lets.distmat(carniv.ras2.data.comp[,c(1,2)])
#Ahora podemos calcular el I de Moran para diferentes clases de distancia y ver el correlograma resultante
carniv.ras2.lm.moran <- lets.correl(residuals(carniv.ras2.lm),carniv.ras2.coords.dist,10)
```
¿Qué tal? ¿Existe autocorrelación? ¿Mucha/Poca? 
¿En cuántas clases de distancia?

##Eigenvector-based Spatial Filters
Paso 1. Crear una matriz de distancias, "truncarla" usando la distancia identificada en correlograma de Moran (¿cuál?) e invertirla para que las distancias cortas tengam más peso

```{r eval=FALSE}
#Crear una matriz de distancia, como un objeto "matriz" en lugar de "dist"
mat.dist <- lets.distmat(carniv.ras2.data.comp[,c(1,2)], asdist = FALSE)

#Generar la matriz truncada, con base en la distancia en la cual ya no tenemos autocorrelación (verificar con el correlograma)
mat.dist.w <- ifelse(test = mat.dist > 677, yes = 4*677, no = mat.dist)

#"Invertir" la matriz para que las distancias cortas tengan más peso: matriz de pesos "w".
#NOTA: este paso no es una "inversión" estadística de la matriz, sino simplesmente una transformación de nuestros valores.
w <- 1/mat.dist.w
diag(w) <- 0

#Corroborar el coeficiente de autocorrelación global (Moran's I) usando la nueva matriz de pesos "w"
Moran.I(residuals(carniv.ras2.lm),w)
```

Paso 2. Hacer el "eigenanalysis" de la matriz de distancias (PCoA: análisis de coordenadas principales)

```{r eval=FALSE}
# PCoA - del paquete "ecodist" (la matriz tiene que ser de tipo "dist")
#NOTA: la función "pco" centraliza la matriz, entonces no hay necesidad de hacerlo antes
#NOTA2: la PCoA se construye a partir de la matriz truncada, no de la matriz de pesos.
carniv.pcord <- pco(as.dist(mat.dist.w))

# Criar um objeto que "salve" os eigenvectors gerados
carniv.vecold <- carniv.pcord$vectors
#quántos vetores? salvar esse número
n <- ncol(carniv.vecold)
```

Paso 3. Selección de eigenvectores: secuencialmente basado en la minimización del índice global de Moran I (verificado antes)

```{r eval=FALSE}
# Crear un "loop" para ir seleccionando los vectores con base en el análisis de autocorrelación espacial de los residuos del modelo global (ambiente + vectores)

for(i in 1:n-1){
  m1 <-lm(carniv.ras2.data.comp$spp_richness~carniv.vecold[,1:i]+carniv.ras2.data.comp$total_precipitation+carniv.ras2.data.comp$precipitation_seazonality+carniv.ras2.data.comp$mean_annual_temperature+carniv.ras2.data.comp$temperature_seazonality)
  res <-m1$residuals		
  mor <- Moran.I(res,w)
  p.mor<-mor$p.value
  if (p.mor>0.05){vec.sel <-(i+2)}
  if (p.mor>0.05){break}
}

#salvar os vetores selecionados
carniv.vecsel <- carniv.vecold[,1:vec.sel]
```
¿Cuántos vectores fueran seleccionados?

##SEVM: spatial eigenvector mapping
Pero, ¿y si lo vemos en el mapa? 
```{r eval=FALSE}
#Rasterizar los vectores seleccionados, solo algunos como ejemplo (el primero y el último; amplia e pequeñaa escala, respectivamente)

#primer vector (amplia escala)
carniv.vecsel1 <- rasterize(carniv.ras2.data.comp[,c(1,2)], carniv.rich, carniv.vecsel[,1])
plot(carniv.vecsel1)

#último vector (pequena escala)
carniv.vecsel64 <- rasterize(carniv.ras2.data.comp[,c(1,2)], carniv.rich, carniv.vecsel[,64])
plot(carniv.vecsel64)
```

¿Cómo se ve el patrón?


Paso 4. Ajustar un modelo incluyendo los vectores espaciales junto con las variables ambientales

```{r eval=FALSE}
carniv.ras2.lm.vecs <- lm(carniv.ras2.data.comp$spp_richness~carniv.ras2.data.comp$total_precipitation+carniv.ras2.data.comp$precipitation_seazonality+carniv.ras2.data.comp$mean_annual_temperature+carniv.ras2.data.comp$temperature_seazonality+carniv.vecsel[,1:19])

summary(carniv.ras2.lm.vecs)
```
¿Qué salió? ¿Explicá máss que el modelo anterior?

Paso 5. Corroborar la autocorrelación en los residuos del nuevo modelo
```{r eval=FALSE}
carniv.ras2.lm.vecs.moran <- lets.correl(residuals(carniv.ras2.lm.vecs),carniv.ras2.coords.dist,10)
```
¿Aún tenemos autocorrelación espacial?

Paso 6. ¿Qué tanta variación en la riqueza es explicada por el ambiente y por el espacio? (partición de varianza)

```{r eval=FALSE}
carniv.varpart <- varpart(Y = carniv.ras2.data.comp$spp_richness, carniv.ras2.data.comp[,c(4:7)], carniv.vecsel)

plot(carniv.varpart)
```
¿Quién explicó más individualmente? ¿y en conjunto?

¿Qué podemos concluir a partir de estos resultados?
