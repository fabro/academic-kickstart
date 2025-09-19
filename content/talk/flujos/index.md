---
title:  "Flujos de Trabajo y organización de proyectos"
author: "Juliana Herrera, Daniel V, Luis D, Fabricio Villalobos"
date: "2025-09-07"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
```

## Paquetes necesarios

-   📦 **skimr**: Resumen de datos
-   📦 **janitor**: Limpieza y tabulación
-   📦 **easystats**: Reportes automáticos


```{r libraries, echo=TRUE, message=FALSE, warning=FALSE}

library(skimr)     # Resumen de datos
library(janitor)   # Limpieza y tabulación
library(easystats) # Reportes automáticos
```

## ¿Qué es R?

-   **R** es un lenguaje de programación y un entorno para análisis estadístico.
-   Libre y de código abierto.
-   Muy utilizado en estadística, ciencia de datos, economía, biología, entre otras áreas.
-   Permite manipulación, análisis y visualización de datos.

## ¿Por qué usar R?

-   Gran colección de paquetes para análisis y visualización.
-   Comunidad activa y mucha documentación.
-   Integración sencilla con otros lenguajes y herramientas (Python, Quarto, etc.).
-   Ideal para reproducibilidad y trabajo colaborativo.

## Entornos de Desarrollo para R (IDEs)

R se puede usar en diferentes entornos de desarrollo, cada uno con características propias:


## R GUI

-   Interfaz gráfica básica que viene al instalar R.
-   Permite ejecutar código y visualizar resultados.
-   Limitada en funcionalidades avanzadas.

## Jupyter Notebooks

-   Permite combinar código, texto y visualizaciones en documentos interactivos.
-   Soporte para R mediante kernels específicos.
-   Muy útil para prototipado y documentación reproducible.

## Positron

-   **Positron** es un entorno de desarrollo para R y Python, creado por Posit (antes RStudio).
-   Ofrece una interfaz moderna y amigable para escribir, ejecutar y depurar código.
-   Permite gestionar proyectos, visualizar datos y generar reportes reproducibles.
-   Integra herramientas colaborativas y extensiones para flujos de trabajo avanzados.

------------------------------------------------------------------------

## RStudio

-   IDE más popular para R.
-   Interfaz amigable, integración con proyectos, scripts, gráficos y paquetes.
-   Herramientas avanzadas para depuración, gestión de proyectos y generación de reportes.
-   [Descargar RStudio](https://posit.co/download/rstudio-desktop/)

## ¿Qué versiones usaremos en este curso?

-   R: 4.3.1 o superior
-   RStudio: 2023.06.1 o superior

## Paquetes en R📦

R base tiene varias funciones por defecto:\
`list()` `log()` `cat()` `rm()`

## Paquetes Adicionales 📦

Se instalan desde R con:

-   `install.packages()` para paquetes de CRAN 📦
-   `remotes` o `pak` para todos los demás repositorios

## Primer aproximación a R y RStudio 📊

-   Consola
-   Ejecutar RStudio
-   Instalación de Paquetes.

``` 
install.packages("tidyverse")
install.packages("ggplot2")
```

## Ayuda

-   `?funcion` o `help(funcion)` para ayuda sobre funciones.
-   `??palabra_clave` para buscar en la documentación.
-   Documentación en línea y foros (Stack Overflow, R-bloggers).
-   Vignetas: Documentos largos que explican el uso de paquetes\
    `vignette(package="package-name")`\
    `browseVignettes("grid")`
-   `?<-` para ayuda sobre operadores.

## Directorios, rutas y nombres en R

-   **Directorio**: Carpeta donde se almacenan archivos y subcarpetas.
-   **Ruta**: Indica la ubicación de un archivo o carpeta dentro del sistema.
    -   **Ruta absoluta**: Comienza desde la raíz del sistema.
        -   Ejemplo: D:/juliana/MacroUdeA2025/datos/datos.csv
    -   **Ruta relativa**: Comienza desde el directorio actual.
        -   Ejemplo: MacroUdeA2025/datos/datos.csv

------------------------------------------------------------------------

### Buenas prácticas de nombres

-   Usa nombres descriptivos para carpetas y archivos.
-   Ejemplo: `proyecto_macro_2025.csv`
-   Evita espacios y caracteres especiales.
-   Mantén una estructura organizada (por ejemplo, separar datos, scripts y resultados).

## Proyectos y carpetas

Para trabajar en R podemos seguir diferentes rutas:\
En lo personal a mí me gusta trabajar con proyectos.

Pero podemos usar el paquete 📦fs


`getwd()`
`setwd()`
`dir_create("proyecto")`
`dir_create("datos")`
`dir_create("scripts")`
`dir_create("resultados")`


## Ejercicio

1.  Crear un proyecto
2.  Crear las carpetas `datos`, `scripts` y `resultados`
3.  Abrir los archivos usando rutas relativas

```{r}

archivo <- read.csv("../datos/penguins_2025.csv")
```

## Primeros pasos en R

-   Todo lo que existe es un objeto
-   Todo lo que ocurre lo hace una función (Las funciones son objetos)
-   Las funciones (generalmente) viven en paquetes

## Tipos de objetos en R

**Variables**

En R, las variables pueden almacenar diferentes tipos de datos:

```{r}
x <- 5
nombre <- "Juliana"
```

## Tipos principales

```{r}
x <- 3.14 # Numéricas: Números reales (decimales)
y <- 10   # Enteras: Números enteros
nombre <- "Juliana" # Caracteres: Texto (strings)
es_mayor <- TRUE # Lógicas: Verdadero o falso
nivel <- factor(c("bajo", "medio", "alto")) # Factores: Categorías o niveles
fecha <- as.Date("2025-09-05") # Fechas: Valores de fecha y hora
```

## Operaciones Básicas

**Operaciones básicas**

```{r}
suma <- 2 + 3
producto <- 4 * 5
```

**Vectores**

```{r}
edades <- c(23, 25, 30)
letras <- c("a", "b", "c")
```

**Data Frames**

```{r}
datos <- data.frame(nombre = c("Ana", "Luis"), edad = c(28, 32))
head(datos)
```

**Matrices**

```{r}
matriz <- matrix(1:6, nrow = 2, ncol = 3)
matriz
```

**Arrays**

```{r}
array_3d <- array(1:24, dim = c(2, 3, 4))
array_3d
```

**Funciones**

```{r}
longitud <- length(edades)
longitud
resumen <- summary(datos)
resumen
```

**Listas**

```{r}
mi_lista <- list(numeros = c(1, 2, 3), letras = c("a", "b", "c"))
mi_lista$numeros
```

## Importar archivos

Podemos realizarlo a través de diferentes paquetes como readr o readxl o con las funciones por defecto:

```{r}

mi_tabla <- read.csv(file = "../datos/penguins_2025.csv",
                     header = TRUE)
```

## Exploración de datos

-   Importar los datos en R

-   Explorar, para:

-   Saber si fueron importados correctamente

-   Resumir características de los datos

-   Limpiar los datos

```{r}

mi_tabla <- read.csv(file = "../datos/penguins_2025.csv",
                     header = TRUE)
```

## `skimr`

Es un paquete con una única función `skim()`.\
La función `skim()` es útil para resumir conjuntos de datos, especialmente grandes conjuntos sin necesidad de explorarlos con detalle (e.g., toda la tabla, línea por línea).\
Es un combo de algunas funciones del R base, como `str()`, `class()`, `summary()`.

## Vamos a aplicar la función summary()

```{r}
summary(mi_tabla)
```

## Vamos a aplicar la función skim()

```{r}

s <- skim(mi_tabla)
s
yank(s, "numeric")
yank(s, "factor")
```

## Preguntas

-   Use la función `skim()` y conteste:
    -   ¿Cuál especie tiene mayor número de registros?
    -   ¿Hay datos faltantes?
    -   ¿Hay datos raros ?

## easystats::report()

```{r}

report(mi_tabla)
model <- lm(bill_len ~ bill_dep, data = mi_tabla)
report(model)
```

## table() vs janitor::tabyl()

```{r}
table(mi_tabla$species) # función default

tabyl(mi_tabla$species) # función del paquete janitor
tabyl(mi_tabla$year)
```

## Tablas con dos variables

```{r}
table(mi_tabla$species, mi_tabla$island) # función default

tabyl(mi_tabla, species, island) %>%
  adorn_percentages() %>%
  adorn_pct_formatting()
```

## Preguntas

-   Al usar la función `adorn_percentages()`, ¿qué porcentajes se están calculando?
-   Hay un argumento llamado denominator = c("all", "row", "col") en la función `adorn_percentages()`. ¿Cuál es la diferencia de usar cada una de las opciones?
-   ¿Cuál es la diferencia entre `adorn_percentages()` y `adorn_pct_formatting()`?
