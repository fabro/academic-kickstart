+++
title = "Ecogeographic Rules"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Reglas Ecogeográficas: el caso de Rapoport

**Macroecología**

Posgrado en Ciencias

Instituto de Ecología, A.C.

## Regla de Rapoport

#### Obtener el atributo de interés: tamaño del área de distribución/range size (usando los datos de los murciélagos del ejercicio de las distribuciones geográficas)

Cargar el paquete `letsR`
```{r eval=FALSE}
library(letsR)
```

Cargar la matriz de presencia-ausencia observada
```{r eval=FALSE}
load("ejercicios_datos/bats_PAM.Rdata")
bats.pam <- m.PAM
```

Calcular el tamaño de 'range' de las especies
```{r eval=FALSE}
bats.ranges <- lets.rangesize(bats.pam, units = "cell")
```

Ahora tenemos una dataframe de una columna con los tamaños de range (número de celdas) de las especies

### ¿Qué enfoque usamos? ¿Interespecífico vs ensamblaje?

#### Interspecific approach

El enfoque interespecífico se basa en el análisis de las especies como unidades, necesitando la caracterización de las propriedades de las especies. Además del atributo (range size), necesitamos obtener la 'ubicación' de las especies en el gradiente (latitud).

Obtener la ubicación de las especies. En este caso, el 'midpoint' de cada especie en el gradiente latitudinal

```{r eval=FALSE}
bats.midpoints <- lets.midpoint(bats.pam, planar = F)
```

Listo! tenemos los datos, ahora poner a prueba la relación latitud~range size
```{r eval=FALSE}
bats.lm <- lm(bats.ranges[, 1] ~ abs(bats.midpoints[, 3]))

summary(bats.lm)
```

Pero, ¿y la cuestión evolutiva? ¿La relación (o falta de) se mantiene?
vamos a ponerlo a prueba!
```{r eval=FALSE}
# cargar los paquetes para el análisis filogenético
library(ape)
library(caper)
# cargar la filogenia de los murciélagos estudiados
bats.tree <- read.tree("ejercicios_datos/bats_tree")
# cambiar los "_" por " " en los nombres de las especies/tips, para que concuerden con los otros datos
bats.tree.spp <- bats.tree$tip.label
bats.tree.spp <- gsub("_", " ", bats.tree.spp)
bats.tree$tip.label <- bats.tree.spp

# juntar los datos
bats.data <- cbind(bats.midpoints[, c(1, 3)], bats.ranges)
names(bats.data) <- c("Species", "MidLat", "Range_size")
# crear un objeto "comparative data" necesario para el análisis, de acuerdo con lo que exige el paquete `caper`
bats.compdata <- comparative.data(bats.tree, bats.data, Species, vcv = T)

# ajustar el modelo de regresión considerando la filogenia (PGLS: phylogenetic generalized least squares)
bats.pgls <- pgls(Range_size ~ abs(MidLat), bats.compdata, lambda = "ML")

summary(bats.pgls)
```
¿Qué tal? ¿Qué resultados obtuvieron?


#### Assemblage approach
El enfoque de ensamblajes se basa en el análisis de los sitios (celdas) como unidades, necesitando la caracterización de las propriedades de los conjuntos de especies que están presentes en esos sitios. Tenemos que calcular una métrica por ensamblaje/celda, como el promedio del valor del atributo para el conjunto de especies, además de la ubicación de la celda (latitud).

```{r eval=FALSE}
# Obtener los valores promedio por ensamblaje en cada celda
# para eso, tenemos el paquete letsR! (shameless selfpromotion, again ;) )
bats.medianRS <- lets.maplizer(bats.pam, bats.ranges, bats.midpoints[, 1], fun = median, ras = T)

# la ubicación de cada celda/sitio ya la tenemos en la PAM (las primeras dos columnas).
# Entonces, podemos tomar únicamente la latitud y probar la relación de la mediana del atributo (range size) y la latitud
bats.lmSites <- lm(bats.medianRS$Matrix[, 3] ~ abs(bats.medianRS$Matrix[, 2]))

summary(bats.lmSites)
```
¿Qué tal? ¿Qué nos dice ese resultado?

Ahora, incluimos la cuestión evolutiva basada en el enfoque PVR (Phylogenetic eigenVector Regression; Diniz-Filho et al. 1998) para estimar los componentes filogenético (P) y específico (S)
```{r eval=FALSE}
# Obtener los componentes P y S, ¿pero cómo? tenemos letsR! Mentira, la función aún no fue incluida en letsR, pero va a estar próximamente. Por ahora, podemos cargar la función versión beta
# cargar los paquetes necesarios para el resto de análisis
library(picante)
library(vegan)
# cargar la función (el archivo .R que debe estar en su carpeta/espacio de trabajo WD)
source("ejercicios_datos/p_s_components.R")

# correr la función
bats.pscomps <- p.s.comps(phy = bats.tree, sppnames = bats.midpoints[, 1], data = bats.ranges, colnum = 1)

# espacializar los componentes
# componente P
bats.meanP <- lets.maplizer(bats.pam, bats.pscomps[[2]][, 1], bats.midpoints[, 1], fun = mean, ras = T)
# componente S
bats.meanS <- lets.maplizer(bats.pam, bats.pscomps[[2]][, 2], bats.midpoints[, 1], fun = mean, ras = T)

# modelos lineales
bats.lm.meanP <- lm(bats.meanP$Matrix[, 3] ~ abs(bats.pam$P[, 2]))

summary(bats.lm.meanP)

bats.lm.meanS <- lm(bats.meanS$Matrix[, 3] ~ abs(bats.pam$P[, 2]))

summary(bats.lm.meanS)
```

Grafiquemos los componentes!
```{r eval=FALSE}
par(mfrow = c(1, 3))
# atributo original
plot(bats.medianRS$Raster, main = "observed range size")
# componente filogenético
plot(bats.meanP$Raster, main = "phylogenetic component")
# componente específico
plot(bats.meanS$Raster, main = "specific component")
```


¿Y ahora? 
¿Qué podemos concluir con esto?
¿Los patrones se mantienen? ¿Por qué sí/no?
