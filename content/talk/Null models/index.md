+++
title = "Null models"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

---
# Modelos nulos en el ensamblaje de comunidades

**Macroecología**

Posgrado en Ciencias

Instituto de Ecología, A.C.
---

#### ¿Cuáles son las preguntas? 
##### 1) ¿Los ensamblajes de especies observados son distintos a muestras aleatorias? "nulo nulo"
##### 2) ¿Los ensamblajes de especies observados son distintos a muestras aleatorias más restringidas? "nulo restringido" (info biológica)


Generar los datos
```{r eval=FALSE}
# vector con el número (e identidad) de géneros presentes en la comunidad regional ("pool" de especies)
generos <- as.vector(c("a", "b", "c", "d", "e", "f"))

# vector con las riquezas observadas en cada comunidad local (e.g. islas)
richness_coms <- as.vector(c(13, 8, 17, 6, 2))

# vector con el número de géneros en cada comunidad local (en el mismo orden que las riquezas observadas)
genrs_coms <- as.vector(c(3, 6, 2, 5, 1))
```

### PASO 1 de los modelos nulos: Calcular el índice de estructura de la(s) comunidad(es)
```{r eval=FALSE}
# crear un vector con los valores de la razón S/G observados en cada comunidad
s_g_obs <- richness_coms / genrs_coms
# calcular el valor promedio de la razón S/G para todas las comunidades
mean_sgo <- mean(s_g_obs)
```

### PASO 2. Crear las comunidades aleatorias
```{r eval=FALSE}
# una comunidad aleatoria con la riqueza de la primer comunidad observada (e.g. richness_coms[1])
c1 <- sample(generos, richness_coms[1], replace = T)
# otra comunidad aleatoria con la riqueza de la segunda comunidad observada (e.g. richness_coms[2])
c2 <- sample(generos, richness_coms[2], replace = T)
# calcular la razón S/G para la primer comunidad aleatoria
sg_null1 <- length(unique(c1)) / genrs_coms[1]
```

#### ¿y tengo que hacer eso para cada una? ¡Qué pereza! 
Intentemos hacerlo más simple:

Crear objetos para salvar resultados
```{r eval=FALSE}
# crear una matriz para "llenarla" con los datos de las comunidades aleatorias
com <- matrix(0, nrow = max(richness_coms), ncol = length(richness_coms))
# crear una matriz para "llenarla" con los datos calculados de las razones S/G de las comunidades aleatorias
sg_nulls <- matrix(0, nrow = length(richness_coms), ncol = 1)
```

### PASO 3. Calcular los índices para cada comunidad. 
```{r eval=FALSE}
# ¿cómo guardamos esos resultados?
# usamos los vectores creados arriba

# crear una matriz para "llenarla" con los datos de las comunidades aleatorias
coms <- matrix(0, nrow = max(richness_coms), ncol = length(richness_coms))
# crear una matriz para "llenarla" con los datos calculados de las razonez S/G de las comunidades aleatorias
sg_nulls <- matrix(0, nrow = length(richness_coms), ncol = 1)

# y los llenamos!

sg_nulls[1, ] <- sg_null1
coms[1:richness_coms[1], 1] <- c1
mean(sg_nulls)
```

#### ¿Y ahora? ¿una por una? NO! vamos a automatizar y ahorrar esfuerzos!!
usamos "loops" en R para repetir todos los cálculos

```{r eval=FALSE}
# crear comunidades "nulas" con muestreo aleatorio del "pool" de especies
for (i in 1:length(richness_coms)) {
  coms[1:richness_coms[i], i] <- sample(generos, richness_coms[i], replace = T)
}

# calcular las razones S/G para cada comunidad local nula e guardarlos
for (i in 1:ncol(coms)) {
  sg_nulls[i, 1] <- richness_coms[i] / length(unique(coms[1:richness_coms[i], i]))
}
```

PERO, esto genera apenas un escenario aleatorio, necesitamos muchos!!!
Entonces, vamos a hacer una función para generar UN escenario nulo y después usar esa función dentro de otra para conseguir iterarla X (simulaciones) veces 

```{r eval=FALSE}
null_sg <- function(generos, richness_coms) {
  # crear una matriz para "llenarla" con los datos de las comunidades aleatorias
  coms <- matrix(0, nrow = max(richness_coms), ncol = length(richness_coms))
  # crear una matriz para "llenarla" con  los datos calculados de las razones S/G de las comunidades aleatorias
  sg_nulls <- matrix(0, nrow = length(richness_coms), ncol = 1)

  # generar las comunidades "nulas"
  for (i in 1:length(richness_coms)) {
    coms[1:richness_coms[i], i] <- sample(generos, richness_coms[i], replace = T)
  }

  # calcular las razones S/G para cada comunidad local nula y guardarlos
  for (i in 1:ncol(coms)) {
    sg_nulls[i, 1] <- richness_coms[i] / length(unique(coms[1:richness_coms[i], i]))
  }

  mean(sg_nulls)
}
```

Ahora, crear otra función que repita a la función anterior y graficar los resultados aleatorias vs el valor observado, para comparar como cualquier otra prueba estadística (de dos colas, valor observados vs distribución teórica, ¿se acuerdan?)

```{r eval=FALSE}
nulls_sg_sim <- function(generos, richness_coms, s_g_obs, sims) {
  sg_means <- matrix(0, nrow = sims, ncol = 1)

  for (i in 1:sims) {
    sg_means[i, 1] <- null_sg(generos, richness_coms)
  }

  if (mean(s_g_obs) > max(sg_means)) {
    hist(sg_means, xlim = c(0, mean(s_g_obs)))
    abline(v = mean(s_g_obs), col = "red")
  } else {
    hist(sg_means, xlim = c(0, max(sg_means)))
    abline(v = mean(s_g_obs), col = "red")
  }
}
```

## Versión restringida (con mayor info biológica)
Modelo nulo restricto: generar un vector con las probabilidades de cada género (manteniendo su orden), basadas en la riqueza de especies observada. 
Esto significa que la probabilidad de muestrear un génerado dado está basada en cuántas especies tiene: más rico en especies, más probable de ser escogido

Generar un vector de probabilidades
```{r eval=FALSE}
prob_sp <- c(1 / 18, 2 / 18, 2 / 18, 3 / 18, 4 / 18, 6 / 18)
```

Cambiar las funciones de arriba para considerar las probabilidades de los generos

```{r eval=FALSE}
nullcons_sg <- function(generos, richness_coms, prob_sp) {
  coms <- matrix(0, nrow = max(richness_coms), ncol = length(richness_coms))
  sg_nulls <- matrix(0, nrow = length(richness_coms), ncol = 1)

  for (i in 1:length(richness_coms)) {
    coms[1:richness_coms[i], i] <- sample(generos, richness_coms[i], replace = T, prob_sp)
  }

  for (i in 1:ncol(coms)) {
    sg_nulls[i, 1] <- richness_coms[i] / length(unique(coms[1:richness_coms[i], i]))
  }


  mean(sg_nulls)
}
```

```{r eval=FALSE}
nullcons_sg_sim <- function(generos, richness_coms, s_g_obs, prob_sp, sims) {
  sg_means <- matrix(0, nrow = sims, ncol = 1)

  for (i in 1:sims) {
    sg_means[i, 1] <- nullcons_sg(generos, richness_coms, prob_sp)
  }

  if (mean(s_g_obs) > max(sg_means)) {
    hist(sg_means, xlim = c(0, (mean(s_g_obs))))
    abline(v = mean(s_g_obs), col = "red")
  } else {
    hist(sg_means, xlim = c(0, max(sg_means)))
    abline(v = mean(s_g_obs), col = "red")
  }
}
```

#### **NOTA**: Tenemos los casos para cada comunidad individual y no solamente para el valor promedio de la razón S/G. Ustedes podrían explorar esos casos individuales
## BUENA SUERTE!!
