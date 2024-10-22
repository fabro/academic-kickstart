+++
title = "Range-Diversity approach"

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Fabricio Villalobos"]

  
+++

# Análisis simultáneo de distribución y diversidad (Range-Diversity)

**Macroecología**

Instituto de Ecología, A.C. - INECOL

A partir de una matriz de presencia-ausencia, que representa información de riqueza y distribución, podemos demostrar la relación intrínseca entre estas dos propiedades fundamentales de la biodiversidad (Arita et al. 2008; 2012). Vamos a demostrarlo!

#### La riqueza promedio proporcional es igual al tamaño de 'range' proporcional.

Generar una matriz "aleatoria"
```{r eval=FALSE}
#¿Cuántas especies (líneas)?
S <- 10
#¿Cuántos sitios (columnas)?
N <- 15
#matriz
m <- matrix(sample(0:1,S*N, replace=TRUE),S,N)
```

Calcular los parámetros básicos de la matriz de presencia-ausencia
```{r eval=FALSE}
#areas de distribución (ranges)
sumrows <- sum(colSums(m))
#riquezas
sumcols <- sum(rowSums(m))
```

La doble sumatoria puede ser cualquiera de las sumas anteriores (vean que la suma de todas las columnas o todos los sitios es LA MISMA COSA!)
```{r eval=FALSE}
sumadupla <- sumrows
```

Obtener el promedio de la riqueza 
```{r eval=FALSE}
meanrich <- sumadupla/N

#promedio proporcional
prop.meanrich <- meanrich/S
```

Obtener el promedio de las distribuciones
```{r eval=FALSE}
meanrange <- sumadupla/S

#promedio proporcional
prop.meanrange <- meanrange/N
```
¿Qué tal? ¿Los valores son iguales/diferentes?

Ahora con datos reales!!! Usaremos los datos y funciones del paquete letsR (shameless selfpromotion ;) )

Cargar el paquete y obtener los datos
```{r eval=FALSE}
library(letsR)

data("Phyllomedusa")
```

Generar la matriz de presencia-ausencia
```{r eval=FALSE}
pam.phyllomedusa <- lets.presab(Phyllomedusa, count = T, xmn = -100, xmx = -20, ymn = -80, ymx = 14)
#dar un vistazo a los argumentos de la función...
```

Graficar el resultado
```{r eval=FALSE}
plot(pam.phyllomedusa)

#tambien da para ver una solamente una especie (e.g. para checar que la distribución esté correcta)
plot(pam.phyllomedusa, name="Phyllomedusa nordestina")
```

Darle un vistazo a la matriz
```{r eval=FALSE}
pam.phyllomedusa$P[1:5,1:5]
```

Repetir los pasos generados con la matriz aleatoria para los datos de Phyllomedusa

**NOTA** Checar el arreglo de la matriz (¿especies en las líneas o en las columnas?), las columnas (¿tienen coordenadas o no?), etc...
