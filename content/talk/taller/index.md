---
title: "Ejercicio Final"
author: "Juliana H"
date: "2025-09-14"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
```

# Apliquemos algo de lo que hemos aprendido

Para ello vamos a basarnos en el artículo **Evolutionary age correlates with range size across plants and animals**. lo puedes encontrar [Aquí](https://www.nature.com/articles/s41467-025-62124-y#Sec9)

Descargamos los datos de:

[Vertlife](https://vertlife.org/phylosubsets/)

[IUCNredlist](https://www.iucnredlist.org/es)

📦 Cargamos los paquetes necesarios


-   📦 **ape**: Análisis filogenético
-   📦 **sf**: Manejo de datos espaciales
-   📦 **ggplot2**: Visualización de datos
-   📦 **rnaturalearth**: Mapas naturales
-   📦 **rnaturalearthdata**: Datos de mapas naturales
-   📦 **tibble**: Estructuras de datos
-   📦 **stringr**: Manipulación de cadenas de texto
-   📦 **dplyr**: Manipulación de datos
-   📦 **tidyr**: Limpieza y organización
-   📦 **readr**: Importación de datos
-   📦 **readxl**: Importación de datos Excel
-   📦 **lme4**: Modelos lineales mixtos
-   📦 **lmerTest**: Pruebas para modelos mixtos
-   📦 **arm**: Herramientas para análisis estadístico
-   📦 **MuMIn**: Modelos multimodelo
-   📦 **ggeffects**: Efectos de modelos


```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=3}
library(ape)
library(sf)
library(ggplot2)
library(rnaturalearth)
library(rnaturalearthdata)
library(tibble)
library(stringr)
library(dplyr)
library(tidyr)
library(readr)
library(readxl)
library(lme4)           
library(lmerTest)        
library(arm)
library(MuMIn) 
library(ggeffects)
```

## Cargar nuestros mapas

Primero vamos a extraer la información de los mapas de nuestro grupo de interés en este caso elegimos el género **Anolis**

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=8}

sp_pol <- st_read("../datos/final/redlist_species_data/data_0.shp")

```

Podemos observar cómo se ven nuestros datos

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=8}
# Primero necesitamos información extra para ubicarnos....
america <- ne_countries(continent = "North America", scale = "medium", returnclass = "sf")
south_america <- ne_countries(continent = "South America", scale = "medium", returnclass = "sf")
fondo_america <- rbind(america, south_america)

```

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=3}
ggplot() +
  geom_sf(data = fondo_america, fill = "lightgray") +
  geom_sf(data = sp_pol, aes(fill = SCI_NAME), alpha = 0.7) +
  theme_minimal() +
  labs(title = "Distribución de Anolis según IUCN") +
  theme(legend.position = "none")

```

Ahora vamos a calcular el área de cada polígono.
Primero debemos usar una proyección adecuada para calcular áreas. Usaremos una proyección Mollweide.

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=8}
sp_pol_proj <- st_transform(sp_pol, crs = "+proj=moll")

# Calcula el área de cada polígono
sp_pol_proj$area <- st_area(sp_pol_proj)

#una tablita con la info de interes
info_pol <- as_tibble(sp_pol_proj) %>% dplyr::select(SCI_NAME, ISLAND, area) %>%mutate(SCI_NAME = str_replace_all(SCI_NAME, " ", "_"),
         areakm = as.numeric(round(area / 1e6, 4)))

```

Ahora vamos a sacar información de los árboles filogenéticos
```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=16}
trees <- read.nexus("../datos/final/tree-pruner-43cca406-8903-42c4-90f2-6053664f2b3a/output.nex")
#podemos plotear un solo arbol 

#Y sacar la info de  la edad de los nodos terminales (edge lengths)
tree<-trees[[1]]
plot(trees[[1]],cex = 0.3)

n <- length(tree$tip.label)
edge_length <- setNames(tree$edge.length[sapply(1:n, function(x,y)which(y==x),y=tree$edge[,2])], tree$tip.label)


```
Pero recordemos que tenemos 100 filogenias. Que son diferentes hipótesis de ancestria

¿Qué podríamos hacer en este caso? 
```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=8}
#  O podemos calcular la edad de los nodos terminales (edge lengths)
#para cada especie en cada árbol y luego promediar esos valores a través de los 100 árboles.

# Obtén el número de especies del primer árbol
n <- length(trees[[1]]$tip.label)
species <- trees[[1]]$tip.label  # Usamos como referencia los nombres de tips

# Matriz para guardar los edge lengths
edge_lengths_tb<- matrix(NA, nrow = n, ncol = 100)
rownames(edge_lengths_tb) <- species

for (i in 1:100) {
  tree_a<- trees[[i]]
  # Calcula los edge lengths para este árbol
  edge_length <- setNames(
    tree_a$edge.length[
      sapply(1:n, function(x, y) which(y == x), y =  tree_a$edge[,2])
    ],
    tree_a$tip.label
  )
  # Guarda los valores en la matriz (alineando por nombre de especie)
  edge_lengths_tb[, i] <- edge_length[species]
}

# Calcula el promedio por especie
edge_mean <- rowMeans(edge_lengths_tb, na.rm = TRUE)

# Resultado: vector nombrado con el promedio del largo terminal por especie
table_ages<-cbind(SCI_NAME=rownames(edge_lengths_tb),edge_mean)
```

Finalmente podemos juntar la info de los mapas con la info de las edades

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=4, fig.height=4}
final<-as_tibble(table_ages)  %>% 
  left_join(info_pol %>% dplyr::select(SCI_NAME, areakm), by="SCI_NAME")%>% 
  mutate(edge_mean=as.numeric(edge_mean)) %>% 
  drop_na() 

  final %>% ggplot(aes(x=edge_mean,y=areakm))+
  geom_point()+
  geom_smooth(method = "lm")+
  labs(x="Edad", y="Area")+
  theme_minimal()

```

Pero ellos trabajaron con escalas logaritmicas 
  
```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=4, fig.height=4}  
  final$logarea<-log10(final$areakm) 
  final$logedge_mean<-log10(final$edge_mean)
  
  final %>% ggplot(aes(x=logedge_mean,y=logarea))+
    geom_point()+
    geom_smooth(method = "lm")+
    labs(x="Edad", y="Area")+
    theme_minimal()
  
```    
    
¿Qué resultados obtuvimos?
¿Cómo podríamos explicarlo?

  
#Ahora vamos a cargar la base de datos del artículo. 
```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=4, fig.height=4}
 
RangeData_raw <- read_xlsx("../datos/final/SpAge_RangeSize/data/EvoRangeAge_DataBase.xlsx")
RangeData_raw$Groups <- as.factor(RangeData_raw$Groups)
RangeData_raw$Order <- as.factor(RangeData_raw$Order)
RangeData_raw$Family <- as.factor(RangeData_raw$Family)
RangeData_raw$Region <- as.factor(RangeData_raw$Region)
RangeData_raw$Island <- as.factor(RangeData_raw$Island)
RangeData_raw$Raw_Ranges <- as.numeric(RangeData_raw$Raw_Ranges)

# Derived variables
RangeData_raw$Ages <- log10(RangeData_raw$Raw_Ages_Median)
RangeData_raw$Ages_corrected <- log10(RangeData_raw$Raw_Ages_MedianCorrected)
RangeData_raw$Ranges <- log10(RangeData_raw$Raw_Ranges)
RangeData_raw$Genus <- str_split(RangeData_raw$Species, "_", simplify = TRUE)

RangeData_raw$Genus <- RangeData_raw$Genus[, 1]
source("../datos/final/SpAge_RangeSize/Scripts/timePeriods.R")
# Filter complete cases for analysis
RangeData <- RangeData_raw[complete.cases(RangeData_raw$Ages), ]


Data <- RangeData[RangeData$Groups == "Reptiles",]

BaseModel_rand <- lmer(Ranges ~ Ages + (1 | Family) , data = Data)
sM_rand <- arm::standardize(BaseModel_rand,standardize.y = TRUE)
summary(sM_rand)
r.squaredGLMM(sM_rand)

xnew <- seq(from = min(Data$Ages,na.rm = TRUE),to = max(Data$Ages,na.rm = TRUE), 
            length.out = 10)

mydf <- ggpredict(
  BaseModel_rand,
  terms = c("Ages[xnew]"),
  ci_level = 0.95,
  type = "random",
  typical = "mean",
  condition = NULL,
  back_transform = TRUE,
  interval = "confidence")

pred_df_mydf <- as.data.frame(mydf)

cbf_1 <- c( "#E69F00", "#56B4E9", "#009E73", 
            "#F0E442", "#0072B2", "#D55E00", "#CC79A7")
```

Y cómo se vería el plot 
```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=4}
 

plot(Ranges ~ Ages, data = Data, pch = 19, col = cbf_1[7], xlim = c(-2.6,2.2),
     ylim = c(-2,9), yaxt = "n", xaxt = "n", xlab = "Species Age (myr)", 
     ylab = "Range size (km2)", main = "Reptiles")

# Add confidence interval envelope
polygon(c(pred_df_mydf$x, rev(pred_df_mydf$x)), c(pred_df_mydf$conf.low, 
                                                  rev(pred_df_mydf$conf.high)), col = "#A9A9A940", border = NA)
lines(mydf$x,mydf$predicted,col = "black", lwd = 2, lty = 1)

axis(1, labels = round(10^c(-2.7,0.0,0.5,1.0,1.5,2.0), digits = 1),
     at = c(-2.7,0.0,0.5,1.0,1.5,2.0),
     col = "black",        # Axis line color
     col.ticks = "black", # Ticks color
     col.axis = "black") 
axis(2, labels = round(10^c(-0,2,4,6,8), digits = 1),
     at = c(-0,2,4,6,8),
     col = "black",        # Axis line color
     col.ticks = "black", # Ticks color
     col.axis = "black") 
tscales.period(-1.2,-1.7,-1.2)
box()

``` 

Solo por curiosidad quiero superponer los datos que obtuvimos con Anolis

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=4}
 

plot(Ranges ~ Ages, data = Data, pch = 19, col = cbf_1[7], xlim = c(-2.6,2.2),
     ylim = c(-2,9), yaxt = "n", xaxt = "n", xlab = "Species Age (myr)", 
     ylab = "Range size (km2)", main = "Reptiles")

# Add confidence interval envelope
polygon(c(pred_df_mydf$x, rev(pred_df_mydf$x)), c(pred_df_mydf$conf.low, 
                                                  rev(pred_df_mydf$conf.high)), col = "#A9A9A940", border = NA)
lines(mydf$x,mydf$predicted,col = "black", lwd = 2, lty = 1)


points(final$logedge_mean, final$logarea, pch = 19, col = "blue", cex = 1)

modelo <- lm(logarea ~ logedge_mean, data = final) # Ajusta el modelo lineal
x_nueva <- seq(-1, 2, length.out = 100)

# Predicción sobre ese rango
y_nueva <- predict(modelo,newdata = data.frame(logedge_mean = x_nueva))

# Dibujar la línea solo en ese rango
lines(x_nueva, y_nueva, col = "black", lwd = 2, lty = 2)


axis(1, labels = round(10^c(-2.7,0.0,0.5,1.0,1.5,2.0), digits = 1),
     at = c(-2.7,0.0,0.5,1.0,1.5,2.0),
     col = "black",        # Axis line color
     col.ticks = "black", # Ticks color
     col.axis = "black") 
axis(2, labels = round(10^c(-0,2,4,6,8), digits = 1),
     at = c(-0,2,4,6,8),
     col = "black",        # Axis line color
     col.ticks = "black", # Ticks color
     col.axis = "black") 
tscales.period(-1.2,-1.7,-1.2)

box()

```

¿Qué conclusiones podemos sacar , desde los resultados y leyendo el artículo?
