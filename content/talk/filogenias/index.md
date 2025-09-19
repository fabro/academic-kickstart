---
title: "Filogenias"
author: "Juliana H"
date: "2025-09-15"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Primero, ¿Qué es un árbol Filogenético? 🌲

-   Un árbol filogenético es un diagrama que muestra la relación de parentesco entre especies (o taxones)

-Los árboles filogenéticos permiten visualizar cómo diferentes especies se relacionan y han divergido a lo largo del tiempo.

-   Uno de los formatos más utilizados para representar filogenias es el *Newick*(.tree)

-   El formato Newick es una notación para representar parentescos utilizando paréntesis y comas.


## Ejemplo 
Si queremos decir que A y B se parecen más entre ellos que con X, lo anotaríamos así:

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| eval: true
#| echo: true

"((A,B),X);"
```
Para que *R* pueda identificar esta notación, se puede usar el paquete `ape`:

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| echo: true
#| eval: true

library(ape)
library(dplyr)
read.tree(text="((A,B),X);") %>% plot()
```

## El formato Newick: 🐕🐺

Es un forrmato anidado, por lo que podemos explorar la relación entre diferentes grupos. Por ejemplo:

Sabemos que los lobos, los coyotes y los zorros pertenecen a Canidae, pero que los lobos y los coyotes son del mismo género

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| eval: true
#| echo: true
Canidae <-"(Vulpes_vulpes,(Canis_lupus,Canis_latrans));"
read.tree(text=Canidae) %>% plot()

```

Si agregamos murciélagos: 🦇

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| echo: true
#| eval: true
#| label: murcis


Mammalia <-"(Desmodus_rotundus,(Vulpes_vulpes,(Canis_lupus,Canis_latrans)));"
read.tree(text=Mammalia) %>% plot()
```


También podríamos anexar otros grupos pj: cuervos y zanates. ¿Estos dónde irían? 🐦

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| label: zan
#| echo: true
#| eval: true

tetrapoda<-"((Quiscalus_mexicanus,Corvus_corax),(Desmodus_rotundus,(Vulpes_vulpes,(Canis_lupus,Canis_latrans))));"

read.tree(text=tetrapoda) %>% plot()
```

Si agregamos una planta, los mamíferos y las aves se anidarían.🌱

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| eval: true
#| echo: true


Eukarya<-"(Sideroxylon_celastrinum,((Quiscalus_mexicanus,Corvus_corax),(Desmodus_rotundus,(Vulpes_vulpes,(Canis_lupus,Canis_latrans)))));"

Nktree<-read.tree(text=Eukarya) 
plot(Nktree)
```

# Partes de una filogenia:🌲

**Conceptos básicos:** - **Nodo:** Punto de divergencia en el árbol. - **Rama:** Conexión entre nodos, representa la evolución. - **Hoja o terminal:** Especie o taxón actual. - **Raíz:** Ancestro común.

```{r, echo=TRUE, message=FALSE, warning=FALSE, fig.width=5, fig.height=5}
#| eval: true
#| echo: true


# Grafica el árbol
plot(Nktree, main="Árbol filogenético Eukarya", cex=0.9)

# Identificar y graficar la raíz
nodelabels("Raíz", node=8, frame="circle", bg="lightblue", col="blue", adj=0)

# Identificar y graficar las hojas (especies terminales)
tiplabels("Tip/Hoja", frame="none", bg="white", col="darkgreen", adj=c(-0.5,-0.5))

# Identificar y graficar los nodos internos (puntos de divergencia)
nodelabels(text = rep("*", 5), node = 9:13, bg = "yellow", frame = "circle", cex = 1)



# Identificar y graficar las ramas (tramos entre nodos)
# Para resaltar una rama, puedes usar edgelabels. Ejemplo mostrando la rama entre los nodos 8 y 9:
edgelabels("Rama", frame="none",edge=3,  col="black", adj=c(-0.5,-0.5))

```

## Mega-filogenias 

Hasta este punto hemos construido de manera intuitiva un árbol filogenético, estableciendo relaciones con base en el conocimiento disponible. Sin embargo, el creciente número de secuenciaciones, junto con los avances en el estudio biológico de las especies y en el registro fósil, ha dado lugar a múltiples propuestas filogenéticas en la literatura. Estas, además, pueden ser exploradas y manipuladas mediante el uso de R.

Aqui algunos ejemplos:

[Hongos](https://www.nature.com/articles/s41559-019-0834-1)🍄
[Anfibios](https://www.nature.com/articles/s41559-018-0515-5)🐸 [Peces](https://fishtreeoflife.org) 🐟

## Practica 

En esta **práctica** haremos uso del paquete `ggtree` para graficar, personalizar y anotar árboles filogenéticos

```{r}
#| eval: false
#| echo: true
#| message: false
#| warning: false

#install.packages("BiocManager")
#BiocManager::install("ggtree")
 
```

## Paquetes necesarios:

```{r}
#| eval: true
#| echo: true
#| message: false
#| warning: false

library(ape)
library(ggtree)
library(deeptime)
library(tidytree)
library(ggimage)
library(geiger)
library(caper)
library(TDbook)
library(here)

```

## ¿Qué insumos utilizaremos?

En este ejercicio utilizaremos una filogenia con las especies del Orden Siluriformes (peces) a las cuales se le conoce su tipo de migración. Esto es un subset de la Megafilogenia para peces propuesta por Rabosky (2018)

```{r}
#| eval: true
#| echo: true

fishdata<-read.table("../datos/DataFish.txt")

fishtree <-read.tree("../datos/treefish.txt")
fishtree$tip.label<-gsub("_", " ", fishtree$tip.label)


```

## Visualizar

Podemos visualizar la información rápidamente usando `ape`,

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true

plot(fishtree, show.tip.label=F)
```

## 📦 `ggtree`

Ahora graficaremos el mismo árbol utilizando ggtree, el cual sigue una formula idéntica a la de `ggplot`. Nuestro árbol base se llamará **p1**:

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true

p1<-ggtree(fishtree,color="black",size=0.5)
plot(p1)
```


Con **p1,** al igual que con cualquier otro gráfico de `ggplot`, podemos agregar parámetros gráficos para personalizar o agregar anotaciones a nuestra filogenia.

Por ejemplo, podemos cambiar la disposición de la filogenia utilizando el parámetro `layout_`:



```{r}
#| eval: true
#| echo: true

p1+layout_dendrogram()+
p1+layout_fan()+
p1+layout_circular()

```

## Color

También podemos cambiar el color del fondo del gráfico

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+  theme_tree(bgcolor="skyblue")
```

## Adicionar información a los tips

Agregar nombres a las puntas:

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true

p1+xlim(NA, 110)+geom_tiplab(size=2,color="black",angle=0)
```

##  Puntas

O modificar las puntas:

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+geom_tippoint(size=2,color="red",fill="black",shape=21)
```

## Adicionar información a los nodos

Etiquetas a los nodos:

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true

p1+geom_nodelab(aes(label=node),size=2,color="black", hjust=-0.6)+
  geom_nodelab(aes(subset = (node == 109), label = "*"),color = "red",size=10,hjust=1)

                                                    
```

## Adicionar formas a los nodos

o formas a los nodos:

```{r, fig.width=4, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+geom_nodelab(aes(label=node),size=2,color="black", hjust=-0.8)+ geom_nodelab(aes(subset = (node == 109), label = "*"),color = "red",size=10,hjust=1)+
  geom_nodepoint(size=2,color="red",fill="black",shape=21)
```

## Escalas temporales

Podemos agregar escalas de tiempo con el paquete `deeptime`

```{r, fig.width=4, fig.height=4,fig.align='center'}
#| echo: true
#| eval: false
#| message: false
#| warning: false

py<-revts(p1)

py+ geom_tiplab(size=1,color="black",angle=0)+xlim(-100,30)+ylim(-2,70)+coord_geo(abbrv = T,neg=T,skip=NULL,dat=list("epochs"),size=2)





```



```{r, fig.width=4, fig.height=4,fig.align='center'}
#| echo: false
#| eval: true
#| message: false
#| warning: false

py<-revts(p1)

py+ geom_tiplab(size=1,color="black",angle=0)+xlim(-100,30)+ylim(-2,70)+coord_geo(abbrv = T,neg=T,skip=NULL,dat=list("epochs"),size=2)

```



```{r, fig.width=4, fig.height=4,fig.align='center'}
#| echo: true
#| message: false
#| warning: false
py+theme_tree2()
```

## Los parametros son aditivos

Recordemos que los parámetros son aditivos (con `+` podemos ir agregando uno por uno)

```{r, fig.width=8, fig.height=8,fig.align='center'}
#| echo: true
#| eval: false
#| message: false
#| warning: false


p1+ layout_fan(angle = 180)+xlim(NA,150)+geom_tiplab(hjust= -0.2,size=2,color="black")+
  geom_tippoint(size=1,color="black",shape=10)+
  geom_nodelab(size=1,color="black")+
  geom_nodepoint(size=2,color="red",alpha=0.3,shape=16)
```



```{r, fig.width=8, fig.height=8,fig.align='center'}
#| echo: false
#| eval: true
#| message: false
#| warning: false


p1+ layout_fan(angle = 180)+xlim(NA,150)+geom_tiplab(hjust= -0.2,size=2,color="black")+
  geom_tippoint(size=1,color="black",shape=10)+
  geom_nodelab(size=1,color="black")+
  geom_nodepoint(size=2,color="red",alpha=0.3,shape=16)

```

## Anotaciones en Árboles

Podemos incluir texto adicional en los gráficos,Por ejemplo: quiero seleccionar la parte de la filogenia donde se encuentra las especies de la Familia *Pimelodidae*

## Obtener nodos

Para esto primero debemos encontrar el nodo del ancestro común más reciente para este grupo (Usando un *tibble* y la estructura de `dplyr` es muy fácil).

Después usando la función `getMRCA()` podemos encontrar el ancestro en común de estas especies

```{r}
#| eval: true
#| echo: true
fishdata$sciname<-row.names(fishdata)
fishdata$sciname <- gsub("_", " ", fishdata$sciname)
pimelo<-fishdata %>% filter(Family=="Pimelodidae")

nodepimelo<-getMRCA(fishtree,pimelo$sciname)
nodepimelo
```



Ahora conociendo que el ancestro en común de la familia *Pimelodidae* se encuentra en el nodo 80, puedo utilizar esta información para anotar la filogenia utilizando el parámetro *geom_cladelab*:

```{r, fig.width=8, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true
p1+xlim(NA, 110)+ geom_cladelab(node=nodepimelo,label = "Pimelodidae",offset=0,barcolor="red",textcolor="red",angle=0, offset.text=0.1)



```



Si queremos resaltar varias familias:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
Siluridae<-fishdata %>% filter(Family=="Siluridae") %>% dplyr::select(sciname)
nodesiluri<-getMRCA (fishtree,Siluridae$sciname)
Loricariidae<-fishdata %>% filter(Family=="Loricariidae") %>% dplyr::select(sciname)
nodelori<-getMRCA (fishtree,Loricariidae$sciname)
Bagriidae<-fishdata %>% filter(Family=="Bagridae") %>% dplyr::select(sciname)
nodeBagri<-getMRCA (fishtree,Bagriidae$sciname)

nodesiluri
nodelori
nodeBagri
```

## parámetros gráficos

Ahora podemos jugar con varios parámetros gráficos

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
#| 
p1+xlim(NA, 110)+
  geom_cladelab(node=nodepimelo,label ="Pimelodidae",offset=0,barcolor="#93C83E",textcolor="#93C83E",angle=0,offset.text=0.1,fontsize=4)+
  geom_cladelab(node=nodesiluri,label="Siluridae",offset=0,barcolor="#007BA7",textcolor="#007BA7",angle=0,offset.text=0.1,fontsize=4)+
  geom_cladelab(node=nodelori,label="Loricariidae",offset=0,barcolor="#AD5691",textcolor="#AD5691",angle=0,offset.text=0.1,fontsize=4)+
  geom_cladelab(node=nodeBagri,label = "Bagridae",offset=0,barcolor="#EE3224",textcolor="#EE3224",angle=0, offset.text=0.1,fontsize=4)
```

## Anotaciones

O incluso anotar grupos sin ancestro común utilizando el parámetro `geom_strip()`:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+xlim(0,120)+
  geom_strip("Neoarius graeffei","Gogo arcuatus", label=" Un clado polifilético", barsize = 2, offset.text = 0.2)
```

## Resaltar

Otro parámetro de anotación muy interesante es *geom_highlight*, el cual nos permite destacar clados, utilizando los nodos de ancestro en común:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+ geom_highlight(node=80,alpha=0.5,fill="#93C83E",type = "rect")
```

¿Qué clado es este? ...



```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1+
  geom_highlight(node=80,alpha=0.5,fill="#93C83E",type = "rect")+
  geom_cladelab(node=80,label = "Pimelodiade",offset=0,barcolor="#93C83E",textcolor="#93C83E", offset.text=0)+
  xlim(0,110)
```

## 📦 `Phylopic`

En `ggtree` como en `ggplot` podemos incluir imagenes de recursos en línea como *phylopic* o enriquecerlas con imágenes propias.

Para poder hacer uso de esta función, primero debemos cargar 2 paquetes extra:

```{r}
#| eval: true
#| echo: true
library("rsvg")
library("rphylopic")
```
 

Probemos con una especie de nuestro interes:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| echo: true
img <- pick_phylopic(name = "Ambystoma mexicanum", n = 1, view = 1)


```

Pueden explorar esta info: [rphylopic viñeta](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.14221){preview-link="true"}

## Guardar siluetas

```{r}
#| echo: true

uuid <- get_uuid(name = "Erythromma najas")# obtener el identificador unico de la imagen
img <- get_phylopic(uuid = uuid,format ="vector" ) #obtener imagen
img<-recolor_phylopic(img,0.7, "blue",fill="black")
plot(img)

save_phylopic(img = img, path = "Cetops.png",
              width = 500, height = 500)
```


Para hacer uso de las imagenes en nuestra filogenia, primero debemos hacer una tabla con los nodos, el nombre de la especie o clado y el phylopic_id.

En este ejemplo utilizaré los clados *Pimelodidae*, *Siluridae*, *Loricariidae*.

Encontrar los phylopic_id es fácil usando la función `phylopic_uid()`

```{r}
#| eval: true
#| echo: true
ids<-phylopic_uid(c("Phractocephalus hemioliopterus","Siluridae","Ameiurus","Chaetostoma microps"))

```



Con estos ids, ya podemos crear nuestra tabla con los datos necesarios y después gráficarlos:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
dt<-data.frame(node=c(nodepimelo,nodesiluri,nodeBagri,nodelori),
               image=ids$uid,Family=c("Pimelodidae","Siluridae","Bagridae","Loricariidae"),
               colorss=c("#93C83E","#007BA7","#EE3224","#AD5691"),
               size=c(0.1,0.1,0.1,0.05))
dt
```


Por ejemplo:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
my_colors <- setNames(dt$colorss, dt$Family)

p1+ylim(-10,80)+geom_cladelab(data = dt,mapping = aes(node = node, label = Family,image = image, color = Family ),geom = "phylopic",offset.text=4,  imagesize=dt$size,)+scale_color_manual(values = my_colors)



```



También podemos combinar otros parametros que ya conociamos:

```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true
p1 + ylim(-10, 80) +geom_cladelab(data = dt,mapping = aes(node = node,label = Family, image = image,color = Family,size = size),
    geom = "phylopic",offset.text = 4 ) +
  scale_size_identity() +
  geom_highlight( data = dt,
    mapping = aes(node = node,
      fill = Family,colour = Family),type = "rect")+scale_color_manual(values = my_colors) +scale_fill_manual(values = my_colors) 


```


```{r, fig.width=8, fig.height=5,fig.align='center'}
#| eval: true
#| echo: true

p2<-ggtree(fishtree,layout="circular",color="black")

p2 + ylim(-10, 80) +geom_cladelab(data = dt,mapping = aes(node = node,label = Family, image = image,color = Family,size = size),
    geom = "phylopic",offset.text =  6) +
  scale_size_identity() +
  geom_highlight( data = dt, mapping = aes(node = node,fill = Family,colour = Family),type = "rect")+
  scale_color_manual(values = my_colors) +scale_fill_manual(values = my_colors) 

```

## Atributos de las especies

También se puede usar `ggtree`, para graficar atributos de las especies en la filogenia.

Para hacer esto, primero debemos cargar los atributos, en este caso usaremos la longitud estándar máxima y el hábito migratorio:

```{r}
#| eval: true
#| echo: true

fishdata<-read.table("../datos/DataFish.txt")
fishdata$label<-gsub("_"," ",fishdata$label)

head(fishdata)
```


Una vez cargados los datos, la manera más fácil de utilizarlos es uniendolos a la filogenia usando la función `full_join`, es importante que las especies estén etiquetadas como label, para que la función las reconozca:

```{r}
#| eval: true
#| echo: true

datatree<-left_join(fishtree,fishdata,by="label")

```

## ¡Listo!

Ahora tenemos una filogenia con atributos y podemos graficarlos juntos

Primero graficaremos los valores continuos de la longitud sobre las puntas del árbol en una escala de colores:

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true

p4<-ggtree(datatree)
  p4+geom_tippoint(aes(color= Length),size=3)
```

## Escala

podemos también personalizar esta escala:

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true

p4+
  geom_tippoint(aes(color= Length),shape=19,size=3)+scale_colour_gradient(low='yellow', high='red',breaks= c(15,50,100,150,200,300))
```


¿Cómo se verían los datos discretos?

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true

p4+geom_tippoint(aes(color=salinity),shape=15,size=4)+ scale_colour_manual(values = c("blue","gray","skyblue"))
```



¿Podemos incluir mas de un dato?

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true

px<- p4+
  scale_color_manual(values = c("blue","gray","skyblue"))+
  scale_fill_manual(values = c("blue","gray","skyblue"))

dd<-data.frame(id=p4$data$label,value=p4$data$Length)  
  
px+ geom_facet(panel="Length",data =dd[1:65,],geom=geom_col, 
               mapping=aes(x=Length,color=salinity,fill=salinity),orientation='y')+
    theme_tree2()
```

## 📦 `ggtreeExtra`

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| echo: true
#| eval: false
library(ggtreeExtra)

p5<-p4+geom_tippoint(aes(colour=AnaCat))+scale_color_manual(values = c("#3d5a80","#98c1d9","#e0fbfc","#293241","#ee6c4d"))

p5+geom_fruit(data=dd[1:65,],
         geom=geom_bar,
         mapping = aes(
                     y=id,
                     x=value,
                     group=salinity,
                     fill=salinity),pwidth=0.38, 
                    orientation="y", 
                    stat="identity",axis.params=list(
                         axis       = "x",
                         text.size  = 1.8,
                         hjust      = 1,
                         vjust      = 0.5,
                         nbreak     = 3))+ scale_fill_manual(values = c("blue","gray","skyblue"))
  

```



```{r, fig.width=6, fig.height=8,fig.align='center'}
#| echo: false
#| eval: true


library(ggtreeExtra)

p5<-p4+geom_tippoint(aes(colour=AnaCat))+scale_color_manual(values = c("#3d5a80","#98c1d9","#e0fbfc","#293241","#ee6c4d"))

p5+geom_fruit(data=dd[1:65,],
         geom=geom_bar,
         mapping = aes(
                     y=id,
                     x=value,
                     group=salinity,
                     fill=salinity),pwidth=0.38, 
                    orientation="y", 
                    stat="identity",axis.params=list(
                         axis       = "x",
                         text.size  = 1.8,
                         hjust      = 1,
                         vjust      = 0.5,
                         nbreak     = 3))+ scale_fill_manual(values = c("blue","gray","skyblue"))
  
```

## Imágenes propias

En este ejemplo usaremos las familias *Pimelodidae*,*Doradidae* y *Loricariidae*

Primero recuperamos un representante de cada grupo, `ape` tiene una función que nos puede ser útil: `keep.tip()`:

```{r, fig.width=6, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true
grouptree<-fishtree %>%
keep.tip(c("Pimelodus maculatus", "Lithodoras dorsalis","Loricariichthys anus"))

grouptree$tip.label<-c("Pimelodidae", "Doraridae","Loricariidae")

p6<-ggtree(grouptree,size=1)+xlim(0,120)+ geom_tiplab(color="navyblue",offset = 0.5,size=5)
p6
```


Para visualizar las imágenes como tips, el archivo, debe tener el nombre exacto del grupo o especie y todas las imaágenes el mismo formato (i.e JPG) :

Las fotos aqui presentadas provienen del Proyecto `cavfish`

```{r, fig.width=10, fig.height=8,fig.align='center'}
#| eval: true
#| include: true
p6+ 
  xlim(NA, 150) +
  geom_tiplab(aes(image=paste0("../datos/img/", label, '.png')),geom="image", offset=3, align=1, size=0.6)+
  geom_tiplab(geom="label",color="black",offset = 30)
```


Hasta ahora hemos usado filogenias conocidas. Pero, ¿Qué pasa si no tengo información de la filogenia del grupo que estoy estudiando?

```{r}
#| eval: true
#| include: true

picuid<-phylopic_uid(c("Argania","Quiscalus","Corvus","Desmodus","Vulpes","Canis lupus","Canis latrans"),seed=123)
pics<-data.frame(label=Nktree$tip.label,uid=picuid$uid)
phylopicNK<-ggtree(Nktree) %<+% pics+
  geom_tiplab(aes(image=uid,color="black"),geom="phylopic",size=0.15)+
  theme(legend.position="none")+
  scale_color_manual(values=c("black"))
```

¿Se acuerdan de esta filogenia?

```{r}
#| eval: true
#| echo: false
phylopicNK
```

## 📦 `rotl`

Desde *Open tree of Life*(OTL), podemos obtener una filogenia utilizando las especies de nuestro interes


OTL tiene un apquete para R (`rotl`) que podemos usar para obtener filogenias

```{r}
#| eval: true
#| echo: true 
#| warning: false

library("rotl")
otl_tree<-rotl::tnrs_match_names(c("Quiscalus mexicanus","Canis lupus","Corvus corax","Canis latrans","Vulpes vulpes","Desmodus rotundus","Sideroxylon celastrinum")) %>% 
  pull(ott_id) %>% 
  tol_induced_subtree(label_format="name")
```


```{r}
#| eval: true
#| echo: true
p_ott<-ggtree(otl_tree)+
  xlim(0,9)+
  geom_tiplab(color="blue")
p_ott
```



Para la función de `rtol`, **NO** es necesario introducir las especies en orden, ya que ésta recuperará la taxonomia válida y nos arrojará una filogenia.

En este caso, OTL nos regresó exactamente la misma filogenia que creamos al principio:

```{r, fig.width=10, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true
p_nk<-ggtree(Nktree)+
  xlim(0,9)+
  geom_tiplab(color="red")

p_ott/p_nk
```



Ahora podemos crear nuestras propias figuras usando estos recursos.

## Ejercicio:

Descargar una filogenia para un grupo de su interes desde OTL y plotearla usando los argumentos vistos y observados.

## ¿Qué info podemos sacar de un árbol filogenético?

```{r, fig.width=10, fig.height=8,fig.align='center'}
#| eval: true
#| echo: true
#| 
library("picante")


fishtree <-read.tree("../datos/treefish.txt")


# Edad total del árbol (desde la raíz hasta las hojas)
tree_age <- max(node.depth.edgelength(fishtree))
head(tree_age)

# Longitud de las ramas terminales.
#funcion tomada de revel phytools
n <- length(fishtree$tip.label)
edge_length <- setNames(fishtree$edge.length[sapply(1:n, function(x,y)which(y==x),y=fishtree$edge[,2])], fishtree$tip.label)

head(edge_length)
# tasas de especiación

equal_splits<-picante::evol.distinct(fishtree, 
type = "equal.splits",
scale = FALSE,
use.branch.lengths = TRUE)

DR<-1/equal_splits$w

tbDR<-cbind(equal_splits$Species,DR)

head(tbDR)

```
