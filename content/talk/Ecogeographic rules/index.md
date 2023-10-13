+++
title = "Ecogeographic Rules"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Regras Ecogeográficas: o caso do Rapoport

**Macroecologia**

Universidade Federal do Rio Grande do Sul
Instituto de Biociências
PPG-Ecologia

## Regla de Rapoport

#### Obtener el atributo de interés: tamaño del área de distribución/range size (usando los datos de los carnívoros del ejercicio de las distribuciones geográficas)

Cargar el paquete `letsR`
```{r eval=FALSE}
library(letsR)
```

Cargar la matriz de presencia-ausencia observada
```{r eval=FALSE}
load("ejercicios_datos/amsur_carniv_PAM.RData")
```

Calcular el tamaño de range de las especies
```{r eval=FALSE}
carniv.ranges  <- lets.rangesize(amsur.carniv.pam, units = "cell")
# #cambiar "_" por " " en los nombres de las especies
# carniv.ranges.spp <- rownames(carniv.ranges)
# carniv.ranges.spp <- gsub(" ","_",carniv.ranges.spp)
# rownames(carniv.ranges) <- carniv.ranges.spp
```
Ahora tenemos un vector con los tamaños de range (número de celdas) de las especies

### ¿Qué enfoque usamos? ¿Interespecífico vs ensamblaje?

#### Interspecific approach
El enfoque interespecífico se basa en el análisis de las especies como unidades, necesitando la caracterización de las propriedades de las especies. Además del atributo (range size), necesitamos obtener la 'ubicación' de las especies en el gradiente (latitud).

Obtener la ubicación de las especies. En este caso, el 'midpoint' de cada especie en el gradiente latitudinal
```{r eval=FALSE}
carniv.midpoints  <-lets.midpoint(amsur.carniv.pam, planar = F)
# 
# carniv.midpoints$Species <- gsub(" ","_",carniv.midpoints$Species)
```

Listo! tenemos los datos, ahora poner a prueba la relación latitud~range size
```{r eval=FALSE}
carniv.lm <- lm(carniv.ranges[,1]~abs(carniv.midpoints[,3]))

summary(carniv.lm)
```

Pero, ¿y la cuestión evolutiva? ¿La relación (o falta de) se mantiene?
vamos a ponerlo a prueba!
```{r eval=FALSE}
#cargar los paquetes para el análisis filogenético
library(ape)
library(caper)
#cargar la filogenia de los carnívoros estudiados
carniv.tree <- read.tree("ejercicios_datos/amsur_carniv_phylo.txt")

carniv.tree$tip.label <- gsub("_"," ",carniv.tree$tip.label)

#prepare the data
carniv.data  <- cbind(carniv.midpoints[,c(1,3)],carniv.ranges)
colnames(carniv.data) <- c("Species","MidLat","Range_size")
#crear un objeto "comparative data" necesario para el análisis, de acuerdo con lo que exige el paquete `caper`
carniv.compdata <- comparative.data(carniv.tree,carniv.data,Species,vcv=T)

#ajustar el modelo de regresión considerando la filogenia (PGLS: phylogenetic generalized least squares)
carniv.pgls <- pgls(Range_size~abs(MidLat),carniv.compdata,lambda='ML')

summary(carniv.pgls)
```
¿Qué tal? ¿Qué resultados obtuvieron?


#### Assemblage approach
El enfoque de ensamblajes se basa en el análisis de los sitios (celdas) como unidades, necesitando la caracterización de las propriedades de los conjuntos de especies que están presentes en esos sitios. Tenemos que calcular una métrica por ensamblaje/celda, como el promedio del valor del atributo para el conjunto de especies, además de la ubicación de la celda (latitud).

```{r eval=FALSE}
#Obtener los valores promedio por ensamblaje en cada celda
#para eso, tenemos el paquete letsR! (shameless selfpromotion, again ;) )
carniv.medianRS <- lets.maplizer(amsur.carniv.pam,carniv.ranges[,1],rownames(carniv.ranges), fun = median, ras=T)

#la ubicación de cada celda/sitio ya la tenemos en la PAM (las primeras dos columnas).
#Entonces, podemos tomar únicamente la latitud y probar la relación de la mediana del atributo (range size) y la latitud
carniv.lmSites  <- lm(carniv.medianRS$Matrix[,3]~abs(carniv.medianRS$Matrix[,2]))

summary(carniv.lmSites)
```
¿Qué tal? ¿Qué nos dice ese resultado?

Ahora, incluimos la cuestión evolutiva basada en el enfoque PVR (Phylogenetic eigenVector Regression; Diniz-Filho et al. 1998) para estimar los componentes filogenético (P) y específico (S)
```{r eval=FALSE}
#Obtener los componentes P y S, ¿pero cómo? tenemos letsR! Mentira, la función aún no fue incluida en letsR, pero va a estar próximamente. Por ahora, podemos cargar la función versión beta
#cargar los paquetes necesarios para el resto de análisis
library(picante)
library(vegan)
#cargar la función (el archivo .R que debe estar en su carpeta/espacio de trabajo WD)
source("ejercicios_datos/p_s_components.R")

#correr la función
carniv.pscomps <- p.s.comps(phy = carniv.tree,sppnames = carniv.midpoints[,1],data = carniv.ranges,colnum = 1)

#espacializar los componentes
#componente P
carniv.meanP <- lets.maplizer(amsur.carniv.pam,carniv.pscomps[[2]][,1],carniv.midpoints[,1], fun = mean, ras=T)
#componente S
carniv.meanS <- lets.maplizer(amsur.carniv.pam,carniv.pscomps[[2]][,2],carniv.midpoints[,1], fun = mean, ras=T)

#modelos lineales
#componente filogenético
carniv.lm.meanP <- lm(carniv.meanP$Matrix[,3]~abs(amsur.carniv.pam$P[,2]))

summary(carniv.lm.meanP)

#componente específico
carniv.lm.meanS <- lm(carniv.meanS$Matrix[,3]~abs(amsur.carniv.pam$P[,2]))

summary(carniv.lm.meanS)
```

Grafiquemos los componentes!
```{r eval=FALSE}
par(mfrow=c(1,3))
#atributo original
plot(carniv.medianRS$Raster, main="observed range size")
#componente filogenético
plot(carniv.meanP$Raster, main="phylogenetic component")
#componente específico
plot(carniv.meanS$Raster, main= "specific component")

```


¿Y ahora? 
¿Qué podemos concluir con esto?
¿Los patrones se mantienen? ¿Por qué sí/no?
