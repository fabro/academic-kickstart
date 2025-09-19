---
title: "Phylopic"
author: "Juliana Herrera, Daniel V, Luis D, Fabricio Villalobos"
date: "2025-09-08"
output: html_document
---

## Phylopic

Es una base de datos que almacena imágenes de siluetas de diferentes organismos. Cada imagen está asociada a uno o más nombres taxonómicos e indica aproximadamente el linaje al que pertenece dicho organismo.

[Phylopic](https://www.phylopic.org/)

[Si quieres saber más](https://theplosblog.plos.org/2015/07/behind-the-scenes-at-phylopic/)

![](images/murci.jpg){.center}

## rPhylopic

rphylopic, un paquete de R para obtener, transformar y visualizar siluetas de organismos de la base de datos PhyloPic.

rphylopic permite a los usuarios modificar la apariencia de estas siluetas para una personalización máxima al programar visualizaciones de alta calidad en flujos de trabajo de R (base) o (ggplot2).

## ¿Por qué?

-   Nuestro compañero Jose Luis ha contribuido a esta gran base de datos con [Siluetas de peces colombianos](https://www.phylopic.org/contributors/81a417eb-0930-44d2-b6ef-c8f0c5a3bf1e/jose-londono-silhouettes)
-   Y todos nosotros podemos contribuir libremente a la construcción de esta base de datos.
-   **Este encuentro tiene como objetivo que conozcamos la herramienta y algunas de sus aplicaciones.**

## Para comenzar

Necesitamos las librerías:

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=8, fig.height=3}
library(rphylopic) 
library(dplyr) 
library(viridis) 
library(ggridges) 
library(ggtree) 
library(ggplot2)
library(rotl) 
library(rphylopic)
library(ape)
library(sf) 
library(rnaturalearth) 
library(rnaturalearthdata) 
library(phytools) 
library(deeptime) 
library(leaflet) 
library(here)
```

[Descarga los datos .csv](https://github.com/JulianaPez/phylopic/blob/master/fishdata.csv)

## Recursos

Aquí vamos a revisar las funciones básicas: [rphylopic](https://github.com/palaeoverse/rphylopic)

Todo lo que vamos a ver está disponible y resumido en la viñeta de los creadores del paquete: [rphylopic viñeta](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.14221)

## Seleccionar siluetas

```{r, echo=TRUE}
# Función para seleccionar de todas las imágenes la que más nos guste
pickp <- pick_phylopic(name = "characidae", n=1)
```

## Guardar siluetas

```{r, echo=TRUE}
uuid <- get_uuid(name = "Erythromma najas") # obtener el identificador único de la imagen
img <- get_phylopic(uuid = uuid, format ="vector") # obtener imagen
img <- recolor_phylopic(img, 0.7, "blue", fill="black")
plot(img)

save_phylopic(img = img, path = here("datos/cetops.png"),
              width = 500, height = 500)
```

## Guardar siluetas

```{r, echo=TRUE}
ps <- ggplot() +
  coord_cartesian(xlim = c(0, 7), ylim = c(0, 7)) + theme_void() +
  add_phylopic(
    img = img,
    x = 3,
    y = 3,
    color = "#8000FF",
    fill = "black",
    alpha = 0.6,
    horizontal = FALSE,
    vertical = FALSE,
    angle = 0,
    ysize = 5
  )
ps
ggsave(ps, file="ps.png")
```

## Plots XY

```{r, echo=TRUE}
fish <- read.csv(here("datos/fishdata_2025.csv")) # leer datos:
uuid <- get_uuid(name = "Sturisomatichthys aureum") # obtener sid
img <- get_phylopic(uuid = uuid, format = "vector") # obtener silueta

plot1 <- fish %>% filter(Specie == "Sturisomatichthys aureum") %>%
  ggplot(aes(x = talla, y = Peso)) +
  geom_point(size = 3, alpha = 0.6, color = "#8000FF") +
  theme_minimal(base_size = 14) +
  labs(title = "Relación entre Talla y Peso", subtitle = "Sturisomatichthys aureum", x = "Talla (cm)", y = "Peso (g)") +
  theme(plot.title = element_text(face = "bold", size = 16, hjust = 0.5), plot.subtitle = element_text(size = 12, hjust = 0.5),
        axis.title = element_text(face = "bold"), panel.grid.minor = element_blank(), axis.line = element_line(size = 0.5, colour = "black", linetype = 1)
  )
plot1
```

## 

Con la silueta

```{r, echo=TRUE}
plot2 <- plot1 + add_phylopic(img = img, x = 12, y = 5, ysize = 7, color = "#8000FF", fill = "black",
  alpha = 0.6, horizontal = FALSE, vertical = FALSE, angle = 0)
plot2
```

## Plots XY con siluetas

```{r, echo=TRUE}
fish %>%
  filter(Specie == "Sturisomatichthys aureum") %>%
  ggplot() + geom_phylopic(
    img = img,
    aes(x = talla, y = Peso, fill = Sexo, size = Peso),
    alpha = 0.8,
    show.legend = TRUE,
    color = "black",
    key_glyph = phylopic_key_glyph(img = img)
  ) +
  theme_minimal(base_size = 16) +
  scale_fill_manual(values = c("#8000FF", "black"), label = c("Hembra", "Macho")) +
  xlim(NA, 18) +
  labs(
    title = "Relación entre Talla y Peso",
    subtitle = "Sturisomatichthys aureum",
    x = "Talla (cm)",
    y = "Peso (g)"
  ) +
  theme(
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    plot.subtitle = element_text(size = 12, hjust = 0.5),
    axis.title = element_text(face = "bold"),
    panel.grid.minor = element_blank(),
    axis.line = element_line(size = 0.5, colour = "black", linetype = 1)
  )
```

## Varias imágenes

Si queremos usar más de una imagen

```{r, echo=TRUE}
# Obtenemos cada una

uuidstu <- get_uuid(name = "Sturisomatichthys aureum")
imgstu <- get_phylopic(uuid = uuidstu, format ="vector")

uuidpim <- get_uuid(name = "Pimelodella macrocephala")
imgpim <- get_phylopic(uuid = uuidpim, format ="vector")

uuidchat <- get_uuid(name = "Chaetostoma")
imgchat <- get_phylopic(uuid = uuidchat, format ="vector")

uuidas <- get_uuid(name = "Astroblepus trifasciatus")
imgas <- get_phylopic(uuid = uuidas, format ="vector")
```

## Gráficos de distribuciones

```{r, echo=TRUE}
plot3 <- fish %>% ggplot(aes(x = Peso, y = Specie, fill = Specie, color = Specie)) +
  geom_density_ridges(alpha = 0.6) + labs(
        title = "Pesos",
        subtitle = "Siluriformes",
        x = "Peso (g)",
        y = "Especies"
  ) + scale_fill_manual(values = c("#440154FF","#31688EFF","#35B779FF","#FDE725FF")) + theme_minimal(base_size = 14) +
  theme(
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    plot.subtitle = element_text(size = 12, hjust = 0.5),
    axis.title = element_text(face = "bold"),
    panel.grid.minor = element_blank(),
    legend.position = "none",
    axis.line = element_line(size = 0.5, colour = "black", linetype = 1)
  ) + add_phylopic(img = imgstu, x = 28, y = 4.2, ysize = 0.5, color = "black", fill = "black", alpha = 1,
               horizontal = FALSE, vertical = FALSE, angle = 0) +
  add_phylopic(img = imgpim, x = 28, y = 3.2, ysize = 0.5, color = "black", fill = "black", alpha = 1,
               horizontal = FALSE, vertical = FALSE, angle = 0) +
  add_phylopic(img = imgchat, x = 28, y = 2.2, ysize = 0.3, color = "black", fill = "black", alpha = 1,
               horizontal = FALSE, vertical = FALSE, angle = 0) +
  add_phylopic(img = imgas, x = 28, y = 1.2, ysize = 0.3, color = "black", fill = "black", alpha = 1,
               horizontal = FALSE, vertical = FALSE, angle = 0)
plot3
```

## Relaciones entre grupos o especies

Cuando no tenemos la filogenia de los grupos podemos recuperar relaciones simples

```{r, echo=TRUE, message=FALSE, warning=FALSE}
nametre <- c("Pimelodella macrocephala", "Chaetostoma leucomelas", "Sturisomatichthys aureum", "Astroblepus trifasciatus")

otl_tree <- rotl::tnrs_match_names(nametre) %>% 
  pull(ott_id) %>% tol_induced_subtree(label_format = "name")

otl_tree$tip.label <- nametre

plot(otl_tree)
```

## Otra forma de recuperar múltiples siluetas

```{r, echo=TRUE, message=FALSE, warning=FALSE}
picuid <- ggimage::phylopic_uid(c("Pimelodella macrocephala", "Chaetostoma", "Sturisomatichthys aureum", "Astroblepus trifasciatus"))
pics <- data.frame(label = otl_tree$tip.label, uid = picuid$uid)
```

## Árboles

```{r, echo=TRUE, message=FALSE, warning=FALSE}
tree0 <- ggtree(otl_tree) %<+% pics + geom_tiplab(offset = 0.5, color = "white") + geom_tiplab(aes(image = uid), geom = "phylopic", size = 0.3, offset = -0.1)
tree0
```

## Árboles

```{r, echo=TRUE, message=FALSE, warning=FALSE}
tree1 <- ggtree(otl_tree, layout = "fan") %<+% pics + geom_tiplab(aes(image = uid), geom = "phylopic", size = 0.8, offset = -0.1)
tree1
```

## Árboles

```{r, echo=TRUE, message=FALSE, warning=FALSE}
treep <- ggtree(otl_tree) + coord_flip()

tree2 <- treep %<+% pics + geom_tiplab(offset = 0.5, color = "white") +
  geom_tiplab(aes(image = uid, color = label), geom = "phylopic", size = 0.3, offset = -0.1) + labs(color = "Especie") +
  scale_color_manual(values = c("#440154FF","#31688EFF","#35B779FF","#FDE725FF"))
tree2
```

## Mapas

```{r, echo=TRUE, message=FALSE, warning=FALSE}
# Muy simple

uuid <- get_uuid(name = "Astroblepus trifasciatus") # obtener el identificador único de la imagen
img <- get_phylopic(uuid = uuid, format ="vector") # obtener imagen
img <- recolor_phylopic(img, 0.7, "black", fill="black")

save_phylopic(img = img, path = here("imagenes/Astro.png"),
              width = 500, height = 500)

world <- ne_countries(scale = "medium", returnclass = "sf")

as <- fish %>% filter(Specie == "Astroblepus trifasciatus") %>% mutate(uid = uuidas) %>% mutate(uu = uuid)

ggplot() + geom_sf(data = world) +
  theme_minimal() +
  labs(title = "Mapa de América del Sur") +
  xlim(-80, -65) + ylim(-5, 15) + ggimage::geom_image(data = as, image = here("imagenes/Astro.png"), aes(x = lon, y = lat), size = 0.05, alpha = 0.75)
```

## Interactivos

```{r, echo=TRUE, message=FALSE, warning=FALSE}
uuid <- get_uuid(name = "Chaetostoma") # obtener el identificador único de la imagen
img <- get_phylopic(uuid = uuid, format ="vector") # obtener imagen
img <- recolor_phylopic(img, 0.7, "black", fill="black")

save_phylopic(img = img, path = here("imagenes/chaeto.png"), width = 500, height = 500)

Cet <- fish %>% filter(Specie == "Chaetostoma leucomelas") 

greenLeafIcon <- makeIcon(
  iconUrl = here("imagenes/chaeto.png"),
  iconWidth = 50, iconHeight = 50,
  iconAnchorX = 50, iconAnchorY = 50
)
leaflet(data = Cet) %>% addTiles() %>%
  addMarkers(~lon, ~lat, icon = greenLeafIcon)
```

## Algunas imágenes que me gustan:

![](images/Burres.jpg){.center}

## Burres et al. 2023

![](images/Burres2.jpg){.center}

## Troyer et al. 2024

![](images/troyer.jpg){.center}
