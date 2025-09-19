---
title: "Visualización de datos en R"
author: "Daniel Valencia, Juliana Herrera, Luis Verde, Fabricio Villalobos"
date: "24-09-2025"
output: html_document
---

> **Objetivo:** Introducir el "lenguaje" de **ggplot2** y practicar generando algunos **gráficos** sencillos (e.g., barras, cajas, puntos...).

## Paquetes para hoy

-   📦 **ggplot2**: visualización de gráficos
-   📦 **dplyr**: manipulación de datos
-   📦 **tidyr**: ordena y reestructura datos (largo/ancho)
-   📦 **patchwork**: ordenar varios gráficos de ggplot2

Por si no los tienen:

```{r libraries, echo=TRUE, message=FALSE, warning=FALSE}
# Instalar si hace falta:
# install.packages(c("ggplot2","dplyr","tidyr","patchwork","viridisLite"))
library(ggplot2)
library(dplyr)
library(tidyr)
library(patchwork)
```

# Introducción a `ggplot2`

**¿Qué es ggplot2?** Es un sistema de graficación en R basado en la *Gramática de los Gráficos*. Permite construir figuras por **capas**, separando datos, estética, geometrías, estadísticas, escalas, coordenadas y temas.

[The Grammar of Graphics](https://link.springer.com/book/10.1007/0-387-28695-0) Wilkinson (1999, 2005)

> Una gramática puede ayudar a construir oraciones diferentes con una pequeña cantidad de verbos, sustantivos y adjetivos, en lugar de memorizar cada oración específica. Una pequeña cantidad de los componentes básicos de ggplot2 y de su gramática puede crear cientos de gráficos diferentes.

[Introducción a la ciencia de datos](http://rafalab.dfci.harvard.edu/dslibro/)\
Rafael A. Irizarry (2020)

**Representaciones gráficas de información.** Pensar en gráficos como combinaciones de **componentes** facilita comunicar patrones; por ejemplo, gradientes, asociaciones tamaño--frecuencia, riqueza por región.

**Gramática de los gráficos (componentes):** - **Datos** (`data`) - **Estética** (`aes()`: x, y, color, fill, size, shape...) - **Geometrías** (`geom_*`) - **Estadísticas** (`stat_*`) - **Escalas** (`scale_*`) - **Coordenadas** (`coord_*`) - **Temas** (`theme()`)

## Generar base de datos para hacer una exploración gráfica

Generamos tres conjuntos de datos simples: - `lagartijas`: *longitud corporal* (mm) y *peso* (g), por ecorregión. - `ranas`: *SVL* (mm) y *frecuencia dominante* (Hz) de cantos, por ecorregión y sexo. (SVL = longitud hocico--cloaca) - `conteo_especies`: riqueza (número de especies) simulada por ecorregión y grupo.

```{r}
set.seed(42)

# Ecorregiones usando un marco geográfico en Colombia:
ecor <- c("Andes","Amazonía","Caribe","Pacífico","Orinoquía")

# 1) Lagartijas: relación longitud–peso (talla–peso)
lagartijas <- tibble(id=1:160,
  ecorregion=sample(ecor, 160, replace=TRUE, prob=c(0.35,0.2,0.15,0.15,0.15)),
  largo_mm=round(rnorm(160, mean=70 + match(ecorregion, ecor)*4, sd=12), 1),
  peso_g=round(0.005 * (largo_mm^2) + rnorm(160, 0, 2), 1)) |> 
  filter(largo_mm > 30, peso_g > 0.5)

# 2) Ranas: SVL vs frecuencia Dominante
ranas <- tibble(id=1:180, ecorregion=sample(ecor,180, replace=TRUE, prob=c(0.4,0.2,0.15,0.1,0.15)),
  sexo=sample(c("Hembra","Macho"), 180, replace=TRUE),
  svl_mm=round(rnorm(180, mean=c(32, 28, 30, 31, 29)[match(ecorregion, ecor)], sd=4), 1)) |>
  mutate(freq_dom_Hz=round(rnorm(n(), mean=4000 - 50*svl_mm + ifelse(sexo=="Macho", 200, 0), sd=250))) |>
  filter(svl_mm > 15, freq_dom_Hz > 800)

# 3) Riqueza por ecorregión y grupo
set.seed(7)

pond_ecor <- c(1.1, 0.9, 1.0, 1.05, 0.95) #Andes, Amazonía, Caribe, Pacífico, Orinoquía

conteo_especies <- expand.grid(ecorregion=ecor, grupo=c("Anfibios","Reptiles","Peces")) |>
  as_tibble() |>
  mutate(riqueza=round(runif(n(),
                          min=c(120, 80, 150)[match(grupo, c("Anfibios","Reptiles","Peces"))] *
                                pond_ecor[match(ecorregion, ecor)] * 0.5,
                          max=c(120, 80, 150)[match(grupo, c("Anfibios","Reptiles","Peces"))] *
                                pond_ecor[match(ecorregion, ecor)] * 1.3)))

dplyr::glimpse(lagartijas); dplyr::glimpse(ranas); conteo_especies
```

## La "Gramática" de los gráficos

```{r}
# objetos geométricos
p_geom <- ggplot() +
  annotate("rect", xmin=0.1, xmax=0.9, ymin=0.15, ymax=0.85, alpha=0.05) +
  annotate("point", x=c(0.2, 0.5, 0.8), y=c(0.2, 0.6, 0.4), size=c(3,5,4), shape=c(21,24,22)) +
  annotate("text", x=0.5, y=0.92, label="Geometrías", fontface="bold") +
  coord_cartesian(xlim=c(0,1), ylim=c(0,1), expand=FALSE) +
  theme_void()

# Escalas y coordenadas
df_axes <- data.frame(x=c(0,1), y=c(0,1))
p_axes <- ggplot(df_axes, aes(x, y)) +
  geom_blank() +
  scale_x_continuous(breaks=seq(0,1,0.25), limits = c(0,1)) +
  scale_y_continuous(breaks = seq(0,1,0.25), limits = c(0,1)) +
  labs(x = "Eje X", y = "Eje Y") +
  theme_classic() +
  ggtitle("Escalas y sistema de coordenadas")

# Combinación (datos + estética + geom + escalas + tema)
p_combo <- ggplot(lagartijas, aes(x=largo_mm, y=peso_g)) +
  geom_point() +
  labs(x="Longitud", y="Peso") +
  theme_minimal()

# Ensamble
(p_geom + p_axes) / p_combo
```

# Personalización de elementos gráficos

## Elementos básicos de personalización

-   `labs()`: títulos, subtítulos, ejes, leyenda, *caption*.
-   `fill` (rellenos), `color` (bordes/líneas), `size` (tamaño), `alpha` (transparencia).
-   `themes` (apariencias predefinidas): `theme_minimal()`, `theme_classic()`, `theme_bw()`, etc.
-   `scale_fill_*`, `scale_color_*`, `scale_shape_*`: control sobre mapeos estéticos.
-   `theme()`: control fino (texto, fondo, líneas, rejilla, leyenda...).

## Paletas de colores y formas (discretas) + mapeos manuales

Generemos un *dataset* con dos variables categóricas para mapear **color** y **shape**.

```{r}
set.seed(1)
paleta_demo <- tibble(x=rnorm(80), y=rnorm(80),
  var1=sample(c("Bosque","Sabana"), 80, replace=TRUE),
  var2=sample(c("Macho","Hembra"), 80, replace=TRUE))

p_pal <- ggplot(paleta_demo, aes(x, y, color=var1, shape=var2)) +
  geom_point(size=3) +
  labs(title="Ejemplo de paletas y formas (discretas)",
       color="Bioma", shape="Sexo")

# Paleta y formas manuales:
p_pal +
  scale_color_manual(values=c("Bosque"="lightskyblue4", "Sabana"="blue")) +
  scale_shape_manual(values=c("Macho"=20, "Hembra"=21)) +
  theme_classic()
```

> **Tip**: Para `shape` 21--25 puedes combinar `fill` (relleno) y `color` (borde).

## mapeo de variables cuantitativas

**Rampa automática** vs **paleta perceptualmente uniforme**:

```{r}
# Rampa automática (gradiente por defecto)
ggplot(ranas, aes(x=svl_mm, y=freq_dom_Hz, fill=svl_mm)) +
  geom_point(size=4, color="white", pch=21) +
  labs(title="Llamados Ranas", 
       x="Tamaño Corporal (SVL)", y="Frecuencia Dominante (Hz)", fill="SVL") +
  theme_minimal()
```

```{r}
# Paleta perceptual (viridis, continua)
ggplot(ranas, aes(x=svl_mm, y=freq_dom_Hz, color=svl_mm)) +
  geom_point(size=3) +
  labs(title="con paleta viridis", 
       x="Tamaño Corporañ (SVL)", y="Frecuencia Dominante (Hz)", color="SVL") +
  scale_color_viridis_c() +
  theme_classic()
```

## Temas (apariencias predeterminadas) y comparación rápida

```{r}
p_base <- 
  ggplot(lagartijas, aes(largo_mm, peso_g, color=ecorregion)) +
  geom_point(alpha=0.85) +
  labs(title="", x="Longitud (mm)", y="Peso (g)", color="Ecorregión")

(p_base + theme_minimal() + theme(legend.position="none")) + (p_base + theme_classic() + theme(legend.position="none")) +
(p_base + theme_bw() + theme(legend.position="none")) + (p_base + theme_light() + theme(legend.position="none"))
```

## Modificar elementos con `theme()`

```{r}
p_base +
  theme_classic(base_size=14) +
  theme(plot.title=element_text(face="bold", hjust=0),
    axis.title=element_text(face="bold"),
    axis.text=element_text(color="gray20"),
    legend.position="bottom",
    legend.title=element_text(face="bold"),
    panel.border=element_rect(color="orange", fill=NA, linewidth=0.7))
```

# Barras, boxplots, violines, histogramas y combinaciones

## Barras: `geom_bar()` (conteos) vs `geom_col()` (valores)

**Conteos automáticos** (p. ej., composición por ecorregión y grupo):

```{r}
ggplot(conteo_especies, aes(x=ecorregion, fill=grupo)) +
  geom_bar(stat="identity", aes(y=riqueza)) +
  labs(title="Riqueza por ecorregión y grupo", x="Ecorregión", y="Riqueza", fill="Grupo") +
  theme_minimal()
```

> *Nota:* Aquí usamos `stat = "identity"` (equivalente a `geom_col()`), porque ya tenemos el valor calculado (`riqueza`).

**Boxplots y violines** para distribuciones por grupos

```{r}
ggplot(ranas, aes(x=ecorregion, y=svl_mm, fill=ecorregion)) +
  geom_boxplot(width=0.65, outlier.alpha=0.7) +
  labs(title="Tamaño corporal por ecorregión", 
       x="Ecorregión", y="Tamaño Corporal (SVL)") +
  theme_classic() +
  theme(legend.position="none")

# ¿nos dicen algo los colores?
```

```{r}
ggplot(ranas, aes(x=ecorregion, y=svl_mm)) +
  geom_violin(trim=FALSE, alpha=0.85) +
  geom_jitter(width=0.12, alpha=0.6) +
  labs(title = "Tamaño Corporal por ecorregión", 
       x="Ecorregión", y="Tamaño Corporal (SVL)") +
  theme_minimal() +
  theme(legend.position="none")
```

**Histogramas y densidades**

```{r}
ggplot(lagartijas, aes(x=largo_mm)) +
  geom_histogram(bins=18, boundary=0, closed="left") +
  labs(title="Distribución de longitud", x="Longitud (mm)", y="Frecuencia") +
  theme_minimal()
```

```{r}
ggplot(lagartijas, aes(x=largo_mm)) +
  geom_density() +
  labs(title="Densidad de longitud", x="Longitud (mm)", y="Densidad") +
  theme_classic()
```

**Dispersión + regresión lineal**

```{r}
ggplot(lagartijas, aes(x=largo_mm, y=peso_g)) +
  geom_point(alpha=0.7, size=3, color="black") +
  geom_smooth(method="lm", se=TRUE, linewidth=1, color="red") +
  labs(title="Peso ~ Longitud", x="Longitud (mm)", y="Peso (g)") +
  theme_minimal()
```

## O de ser el caso podemos hacer combinaciones

**Distribución de longitud + Densidad**

```{r}
ggplot(lagartijas, aes(x=largo_mm)) +
  geom_histogram(aes(y=after_stat(density)), bins=18, boundary=0, closed="left", alpha=0.5) + 
  geom_density(linewidth=1) +
  labs(title="Distribución de longitud + Densidad", x="Longitud (mm)", y="Densidad") +
  theme_minimal()
```

**boxplot + jitter**

```{r}
ggplot(lagartijas, aes(ecorregion, peso_g)) +
  geom_violin(alpha=0.85, width=1, fill="grey")+
  geom_boxplot(width=0.2, outlier.shape=NA) +
  geom_jitter(width=0.12, alpha=0.6) +
  labs(title="Peso por ecorregión", x="Ecorregión", y="Peso (g)") +
  theme_classic() +
  theme(legend.position="none")
```

# Exportando gráficos

## Opciones

-   Botón **Export** de RStudio (rápido para el último gráfico)
-   Dispositivos gráficos base (`png()`, `pdf()`, etc.)
-   `ggsave()` (guarda último gráfico o un objeto `ggplot`)
-   Formatos: **png**, jpg, **pdf** (vectorial), svg

## Exportando con dispositivo gráfico (base R)

Los gráficos se pueden asignar a objetos y luego exportar.

```{r echo=TRUE, message=FALSE, warning=FALSE}
figura_ranas <- ggplot(ranas, aes(x=svl_mm, y=freq_dom_Hz, fill=svl_mm)) +
  geom_point(size=4, color="white", pch=21) +
  labs(title = "Ranas: SVL vs Frecuencia dominante", 
      x="Tamaño Corporal (SVL)", y="Frecuencia Dominante (Hz)", fill="SVL") +
  theme_minimal()

png(filename="Ranas.png", width=12, height=9, units="cm", res=300)
figura_ranas
dev.off()
```

## Dispositivos gráficos de `ragg` (opcional)

-   Más rápidos y con mejor rasterizado

-   Requiere instalar el paquete `ragg`

```{r echo=TRUE, message=FALSE, warning=FALSE}
ragg::agg_png(filename="Ranas_ragg.png", width=12, height=9, units="cm", res=300)
figura_ranas
dev.off()

```

## `ggsave()`

-   Guarda el último gráfico o un objeto `ggplot2`

-   Usa ragg automáticamente si está disponible

```{r}
ggsave(plot=figura_ranas, filename="Ranas_ggsave.jpg",
       width=12, height=9, units="cm", dpi=300)
```

## ¿Quedó raro? todos se ven muy grandes

```{r}
ggsave(plot=figura_ranas, filename="Ranas_ggsave_x2.jpg",
       width=12, height=9, units="cm", dpi=300, scale=2)
```

Argumento `scale` en `ggsave()` y `scaling` en dispositivos de `ragg` para ajustar

## Figuras multipanel

Funciones `facet()`

`facet_wrap()`{style="color:orange" size="1.2em"} para separar por una sola variable y 'envolver'

`facet_grid()`{style="color:orange" size="1.2em"} para separar con combinaciones de variables

```{r}
# facet_wrap: por ecorregión
ggplot(ranas, aes(svl_mm, freq_dom_Hz)) +
  geom_point() +
  facet_wrap(~ecorregion) +
  labs(x="Tamaño Corporal (SVL)", y="Frecuencia Dominante (Hz)")

```

```{r}
# facet_grid: por sexo (filas) y ecorregión (columnas)
ggplot(ranas, aes(svl_mm, freq_dom_Hz, color=sexo)) +
  geom_point(alpha=0.8) +
  facet_grid(rows=vars(sexo), cols=vars(ecorregion)) +
  labs(x="Tamaño Corporal (SVL)", y="Frecuencia Dominante (Hz)", color="Sexo") +
  theme(legend.position="bottom")
```

# Figuras compuestas {background-image="imgs/patchdrone.jpg"}

## `patchwork`

-   Símbolo `+` para acomodar dos gráficos juntos

-   Símbolo `|` para figuras lado a lado

-   Símbolo `/` para apilar figuras

-   Álgebra simple para anidar y acomodar elementos

-   `plot_layout()` para controlar la composición

-   `plot_annotation()` para rotular elementos o agregar textos

------------------------------------------------------------------------

```{r}
ggplot(lagartijas, aes(x=largo_mm, y=peso_g)) +
  geom_point(alpha=0.7, size=2, color="black") +
  geom_smooth(method="lm", se=TRUE, linewidth=0.7, color="red") +
  labs(title="Lagartijas (peso ~ talla)", x="Longitud (mm)", y="Peso (g)") +
  theme_minimal()

ggplot(ranas, aes(svl_mm, freq_dom_Hz, color=svl_mm)) +
  geom_point() + scale_color_viridis_c() +
  labs(title="Ranas: SVL vs frecuencia", x="SVL (mm)", y="Frecuencia (Hz)", color="SVL") +
  theme_minimal()

```

------------------------------------------------------------------------

Más fácil de leer:

```{r}
p_lag <- ggplot(lagartijas, aes(x=largo_mm, y=peso_g)) +
  geom_point(alpha=0.7, size=2, color="black") +
  geom_smooth(method="lm", se=TRUE, linewidth=0.7, color="red") +
  labs(title="Lagartijas", x="Longitud (mm)", y="Peso (g)") +
  theme_minimal()

p_ran <- ggplot(ranas, aes(svl_mm, freq_dom_Hz, color=svl_mm)) +
  geom_point() + scale_color_viridis_c() +
  labs(title="Ranas", x="SVL (mm)", y="Frecuencia (Hz)", color="SVL") +
  theme_minimal()

# Lado a lado
p_lag | p_ran
```

------------------------------------------------------------------------

Qué tal con tres figuras:

```{r}
p_eco_ran <- ggplot(ranas, aes(x=ecorregion, y=svl_mm)) +
  geom_boxplot(width=0.65, outlier.alpha=0.7) +
  labs(title="Ranas ecorregiones", x="Ecorregión", y="Tamaño Corporal (SVL)") +
  theme_classic() +
  theme(legend.position="none")

p_lag + p_ran + p_eco_ran
```

------------------------------------------------------------------------

En una sola columna

```{r}
p_lag + p_ran + p_eco_ran +
  plot_layout(ncol=1)
```

------------------------------------------------------------------------

Anidando:

```{r}
(p_lag + p_ran) / p_eco_ran
```

------------------------------------------------------------------------

```{r}
(p_lag / p_ran) | p_eco_ran
```

------------------------------------------------------------------------

Con anotaciones

```{r}
((p_lag / p_ran) | p_eco_ran)+
  plot_annotation(title="Mi figura de tres elementos",
                  subtitle="Ejemplo de patchwork",
                  tag_levels="A")
```

`tag_levels` genera diferentes secuencias alfabéticas o numéricas para identificar a cada elemento

```{r}
panel <- ((p_lag / p_ran) | p_eco_ran)+
  plot_annotation(tag_levels="A")

ggsave("Compuesto.jpg", panel, scale=1, width=18, height=14, units="cm", dpi=300)
```

## Extensiones para `ggplot2`

[Taller LatinR](https://luisdva.github.io/ggmas/)

[Galería oficial de extensiones](https://exts.ggplot2.tidyverse.org/)
