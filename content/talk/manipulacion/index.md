---
title: "Manipulación y limpieza de datos"
author: "Juliana Herrera, Daniel V, Luis D, Fabricio Villalobos"
date: "2025-09-08"
output: html_document
---
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
```


## Introducción

-   **R** es un lenguaje potente para análisis de datos.
-   El **tidyverse** es un conjunto de paquetes que facilita la manipulación y limpieza de datos.

------------------------------------------------------------------------

## ¿Qué es el tidyverse?🌌

**Tidyverse** es un conjunto de paquetes de R que comparten una misma filosofía y estructuras de datos, diseñados para:

-   Tareas comunes con datos: importar archivos, limpiar datos, transformar, visualizar, o programar nuevas funciones.
-   Facilitar su aprendizaje y que los usuarios vayan aprendiendo más funciones conforme interactúen con más elementos de este ‘ecosistema’.

## Colección de paquetes R para ciencia de datos:

-   📦 **dplyr**: manipulación de datos
-   📦 **tidyr**: limpieza y organización
-   📦 **readr**: importación de datos
-   📦 **ggplot2**: visualización
-   📦 **purrr**: programación funcional
otros paquetes útiles:
-   📦 **ggplot2**: Visualización de datos

```{r libraries, echo=TRUE, message=FALSE, warning=FALSE}
# Carguemos los paquetes que usaremos
library(tidyr)  
library(dplyr)
library(readr)
library(ggplot2)
```

------------------------------------------------------------------------

## Importar datos con readr 📰

```{r}
birds <- read_csv("../datos/medidasaves_2025.csv")
head(birds)
```

------------------------------------------------------------------------

## Introducción a Pipes (`%>%`)

-   El **pipe** (`%>%`) es una herramienta fundamental en el tidyverse.
-   Permite encadenar operaciones de manera clara y legible.
-   Estructuramos operaciones seriadas de izquierda a derecha.
-   Facilita la lectura y comprensión del código.
-   Evitamos creación de variables intermedias. Insertamos con `ctrl + shift + M`.

## Ejemplo

```{r}
# sin pipes
res1 <- birds[which(birds$Department == "Antioquia"),]
res2 <- res1[,c("Species", "Sex")]
head(res2)
```

## 

```{r}
# Con pipes

res <- birds %>%
  filter(Department == "Antioquia") %>%
  select(Species, Sex) %>%
  slice_head(n = 10)
res
```

## Tibbles 📄

-   Son una versión mejorada de los data frames tradicionales de R.
-   Impresión amigable: Al mostrar en consola, solo enseña las primeras filas y columnas, evitando saturar la pantalla.
-   No cambian los tipos de datos automáticamente: Evitan conversiones inesperadas de caracteres a factores.
-   Permiten nombres de columna no estándar: Puedes tener nombres con espacios o símbolos.
-   Mejor integración con tidyverse: Funcionan perfectamente con paquetes como `dplyr`, `tidyr`, `ggplot2`, etc.

## Tidy data 📄

**Datos ordenados** ("tidy data") es un concepto fundamental en el análisis de datos con R y tidyverse.

## Principios de datos ordenados

1.  **Cada variable** se almacena en una columna.
2.  **Cada observación** se almacena en una fila.
3.  **Cada tipo de unidad observacional** forma su propia tabla.

## Ejemplo de datos ordenados 📄

| Locality        | Elevation | Species         | Weight |
|-----------------|-----------|-----------------|--------|
| Valle del Cauca | 3500      | Accipiter stria | 103.36 |
| Antioquia       | 3407      | Adelomyia me    | 4      |
| Quindio         | 2970      | Accipiter stria | 108.33 |

## Ventajas de los datos ordenados 📄

-   Facilitan la manipulación y el análisis.
-   Permiten el uso eficiente de las funciones del tidyverse.
-   Facilitan la visualización y la modelación.
-   Mejoran la reproducibilidad del análisis.

## Consejos para ordenar tablas 💡

-   Todos los valores vacíos deben ser `NA` y no `""` o `0`.
-   Evitar agrupar variables.
-   Usar encabezados útiles.
-   No usar espacios.

## Manipulación de datos básica con 📦 `dplyr`

-   **`filter()`**: Selecciona filas según condiciones lógicas.
-   **`select()`**: Elige columnas específicas del dataframe.
-   **`mutate()`**: Crea o transforma columnas.
-   **`arrange()`**: Ordena filas por una o más variables.
-   **`summarise()`**: Resume/consolida información en el dataframe.
-   **`group_by()`**: Agrupa datos para operaciones posteriores.
-   **`rename()`**: Cambia el nombre de una columna.

## Ejemplo de uso de 📦 funciones en `dplyr`

```{r}
birds_filter1 <- birds %>%
  select(Department, Species, Weight) %>% # Quiero seleccionar solo las columnas Department, Species, Weight
  filter(Department %in% c("Quindio", "Antioquia")) %>% # Quiero filtrar las especies del Amazonas y Antioquia
  group_by(Department, Species) %>% # Quiero agrupar por especie
  drop_na() %>% # Eliminar los NA
  summarise(mean_weight = mean(Weight), n_individuos = n()) %>% # Contar el número de individuos por especie registrados
  arrange(desc(mean_weight)) %>%    # Presentarlas ordenadas de mayor a menor peso
  mutate(mean_weight_mg = mean_weight/1000) %>% # Presentar el peso en mg
  rename(media_peso_mg = mean_weight_mg)
head(birds_filter1)
```

## Otros operadores: 🛠️

Otros operadores importantes que no debemos dejar pasar son:

-   `&` es el operador "y" (AND): Devuelve TRUE solo si ambas condiciones son verdaderas.
-   `|` es el operador "o" (OR): Devuelve TRUE si al menos una condición es verdadera.
-   `!` es el operador "no" (NOT): Invierte el valor lógico (TRUE a FALSE y viceversa).

## Ejemplos con operadores lógicos

¿Qué especies están en Antioquia **Y** por encima de los 2500m?

```{r}
birds %>%
  select(Department, Species, Elevation) %>%
  filter(Department == "Antioquia" & Elevation > 2500) %>%
  group_by(Species) %>%
  summarise(n_individuos = n())
```

## 

¿Qué especies están en Antioquia o en Quindio?

```{r}
birds %>%
  select(Department, Species, Elevation) %>%
  filter(Department == "Antioquia" | Department == "Quindio") %>%
  group_by(Species) %>%
  summarise(n_individuos = n())
```

## Funciones útiles de 📦 tidyr

-   **`separate()`**: Divide una columna en varias columnas.
-   **`unite()`**: Une varias columnas en una sola columna.
-   **`drop_na()`**: Elimina filas con valores NA.
-   **`fill()`**: Rellena valores faltantes con el valor anterior o siguiente.

## Ejemplo

```{r}
birds %>%
  dplyr::select(Species, wingArea) %>%
  separate(col = "Species", into = c("Genus", "Specie"), sep = " ") %>%
  drop_na()

# Ahora probemos fill()
```

## 🍕 Rebanando filas

Familia de funciones slice\_ para obtener máximos, mínimos, muestreos aleatorios, etc.

```{r}
birds %>% select(Species, Weight) %>% slice_max(Weight, n = 5)  # las 5 aves más pesadas

birds %>% select(Species, Weight) %>% slice_min(Weight, n = 5)  # las 5 aves más ligeras

birds %>% select(Species, Weight) %>% slice_sample(n = 5)       # muestra aleatoria de 5 aves
```

## Operaciones entre dos tablas **`_join 🗎+🗎`**

```{r}
taxon <- read_csv("../datos/taxon_info_2025.csv")
birds %>% select(Species, Weight) %>% right_join(taxon)
```

## Otras funciones y argumentos útiles

`!` devuelve el complemento

```{r}
birds %>% select(!c(rightsHolder, basisOfRecord, institutionCode, collectionCode, catalogNumber,
recordedBy, Source, Date))
```

## `where()`

Selecciona las variables para las cuales alguna comparación regrese TRUE

```{r}
birds %>% select(where(is.numeric)) %>% head()
```

## `matches()`

Encuentra nombres de variables con expresiones regulares

```{r}
birds %>% select(matches("tail")) %>% head()
```

## `:`

Selecciona variables contiguas

```{r}
birds %>% select(Species:billDepth) %>% head()
```

## `-`

Otra forma de eliminar variables (columnas)

```{r}
birds %>% select(-c(rightsHolder:Source, Date))
```

# 📦 `stringr`

-   El paquete **stringr** del tidyverse proporciona funciones para manipular cadenas de texto de manera eficiente y coherente.

## Funciones comunes 📦 `stringr`

-   `str_detect()`: Detecta patrones en cadenas.
-   `str_replace()`: Reemplaza partes de cadenas.
-   `str_split()`: Divide cadenas en partes.
-   `str_to_lower()` y `str_to_upper()`: Cambia el caso de las letras.
-   `str_trim()`: Elimina espacios en blanco al inicio y final.

## Formatos de tablas y análisis

Vamos a practicar cómo **reformatear datos** entre formato ancho y largo usando las funciones `pivot_longer()` y `pivot_wider()` del paquete **tidyr**.

## Datos en formato ancho

```{r}
# Datos en formato ancho
birds_ex <- tibble(
  Species = c("Accipiter stria", "Adelomyia me"),
  Weight_2022 = c(103.36, 4),
  Weight_2023 = c(105.00, 4.2)
)

print(birds_ex)
```

## De ancho a largo: `pivot_longer()`

Transforma varias columnas en filas para análisis flexibles.

```{r}
long_birds <- birds_ex %>%
  pivot_longer(cols = starts_with("Weight"),
               names_to = "Year",
               values_to = "Weight")

print(long_birds)
```

## De largo a ancho: `pivot_wider()`

Agrupa filas en columnas para comparaciones directas.

```{r}
wide_birds <- long_birds %>%
  pivot_wider(names_from = Year,
              values_from = Weight)

print(wide_birds)
```

------------------------------------------------------------------------

## Ejercicio

Resumir la tabla birds para saber cuántas especies hay en cada Departamento

```{r}
dep_birds <- birds %>% select(lifeStage, n) %>%
  pivot_wider(names_from = lifeStage,
              values_from = n)

print(dep_birds)
```

## Obtener datos desde paquetes

```{r}
data(package = "ggplot2")

data(diamonds)
head(diamonds)
```
