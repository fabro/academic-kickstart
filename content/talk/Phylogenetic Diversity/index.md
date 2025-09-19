+++
title = "Diversidad Filogenética"

# Authors. Comma separated list.
authors = ["Juliana Herrera Perez"]

  
+++


# Diversidad filogenética y rPD

La diversidad filogenética es calculada como la suma de las longitudes de rama de las especies que co-ocurren en un ensamblaje regional. Los residual obtenidos entre PD y Riqueza es un proxi para identificar eventos evolutivos importantes (especiación, extinción o dispersión) y nos ayudan a inferir como estos han constribuido alos patrones de diversidad que observamos.

```         
library(letsR)
library(terra)
library(dplyr)
library(ggplot2)
library(viridis)
library(patchwork)
library(classInt)
library(scales)

```

Para este ejercicio vamos a cargar el R Data que contiene la PAM y la filogenia que incluye las especies de Carnívoros presentes en sur América. (Ya saben cómo construirla)

```         
load('Datacarn.RData')
```

Para explorar la PAM podemos plotear el raster de riqueza que contiene

```         
pam_mamm$Richness_Raster<- terra::rast('richcarn.tif')
plot (pam_mamm$Richness_Raster)
```

Podríamos hacer algo más **elegante** usango ggplot

```         
SR<-as.data.frame(pam_mamm$Richness_Raster,xy=TRUE,na.rm=TRUE)%>% 
  filter (lyr.1!=0)%>%rename( "Riqueza"="lyr.1")

mapSR<-ggplot()+geom_tile(data = SR , aes(x = x, y = y, fill = Riqueza))+
  scale_fill_viridis_c(limits = c(0, 25),option = "H")+theme_void()+
  theme(axis.text.x=element_blank(),
        axis.ticks.x=element_blank(),
        axis.text.y=element_blank(),
        axis.ticks.y=element_blank())
```

Para el cálculo de PD necesitamos una matriz de presencia ausencia.

```         
pa<-pam_mamm$Presence_and_Absence_Matrix
pas<-pa[,c(-1,-2)]
```

Calculamos PD

```         
#calculamos pd
pdmaam<-picante::pd(pas,phymmam, include.root=FALSE)

#adicionamos una columna a la matriz PA
Pd_table<-cbind(pam_mamm$Presence_and_Absence_Matrix,pdmaam)

#ordenamos la tabla
sort<-Pd_table%>% na.omit()%>% as.data.frame() %>% arrange(desc(SR))

#realizamos una regresion LOESS
model<- loess(PD ~ SR,data=sort)
```

Y ploteamos esta relación

```         
plot(sort$SR,sort$PD, xlab="Riqueza",ylab="PD",pch=19)

pred<-predict(model)
lines(pred, x=sort$SR, col="red")
```

¿Qué observas? ¿Y si lo vemos en mapas?

Primero ploteamos PD

```         
names(sort)[c(1,2)]<-c("x","y")
rPD <- residuals(model)
rPDtable<-cbind(sort,rPD)

#Mapa PD 
mapPD <- ggplot() +
  geom_tile(data = rPDtable, aes(x = x, y = y, fill = PD)) +
  scale_fill_viridis_b(
    breaks = seq(1.6, 503, length.out = 36), # Genera 36 intervalos
    option = "H",                            # Escala viridis H
    labels = c("1.6", rep("", 34), "503")) +   # Etiquetas para el color bar) 
    theme_void() +
      theme(
        axis.text.x = element_blank(),
        axis.ticks.x = element_blank(),
        axis.text.y = element_blank(),
        axis.ticks.y = element_blank()
      )
 
 mapPD 
```

Ahora ploteamos los residuales

```         

positive <- rPDtable$rPD[rPDtable$rPD >= 0]
negative <- rPDtable$rPD[rPDtable$rPD < 0]

# Crear 16 intervalos para positivos y negativos
breaksp <- seq(min(positive), max(positive), length.out = 16)  # 16 intervalos
breaksn <- seq(min(negative), max(negative), length.out = 16)  # 16 intervalos
breakstotal <- c(breaksn, breaksp)

# Crear paletas de colores
colp <- viridis_pal(alpha = 1, option = "rocket", direction = -1)(20)  # Colores para positivos
coln <- viridis_pal(alpha = 1, option = "mako", direction = 1)(20)     # Colores para negativos
cols <- c(coln[1:16], colp[1:16])  # Combinamos las paletas

# Mapeo usando ggplot
maprPD <- ggplot() +
  geom_tile(data = rPDtable, aes(x = x, y = y, fill = rPD)) +
  scale_fill_gradientn(colours = cols, values = scales::rescale(breakstotal)) +
  theme_void() +
  theme(
    axis.text.x = element_blank(),
    axis.ticks.x = element_blank(),
    axis.text.y = element_blank(),
    axis.ticks.y = element_blank()
  )

#Veamos todo Junto
mapSR + mapPD + maprPD
```

¿Qué significan los residuales **positivos** ?¿Y los **negativos**?

Los residuales positivos muestran una mayor diversidad filogenetica de lo esperado por la riqueza. Pueden ser lugares dónde se presentó una baja diversificaccion insitu pero se dieron muchos eventos de dispersión.

Por otro lado los residuales regativos indican coexistencia de linajes recientemente derivados o estrechamente relacionados.

y si queremos ver la riqueza y la diversidad filogenetica al mismo tiempo???

```         

rPDtable 
SR_class  <- classInt::classIntervals(rPDtable$SR,n=5,style = "quantile") 
RPD_class <- classInt::classIntervals(rPDtable$rPD,n=5,style = "quantile")

SR_col  <- classInt::findCols(SR_class) 
RPD_col <- classInt::findCols(RPD_class)

tb <- cbind(rPDtable[,c(1,2)],SR_col,RPD_col)

tb$alpha <- as.character(tb$SR_col + tb$RPD_col) 
tb$color = as.character(atan(tb$RPD_col /tb$SR_col))

Bimap<-ggplot(tb, aes(x = x, y = y)) + geom_tile(aes(fill=color, alpha=alpha)) +
scale_fill_viridis_d(option="inferno")+ 
theme_void() + theme(legend.position = "none") 

leg<- ggplot(expand.grid(x=1:5,y=1:5), aes(x,y))+ geom_tile(aes(alpha=x+y,fill=atan(y/x)))+ 
scale_fill_viridis_c(option="inferno")+ labs(x="SR", y="rPD")+ 
scale_x_continuous(breaks = c(1,5), 
labels = c(0,23)) +
scale_y_continuous(breaks = c(1,5), 
labels = c(-100,60)) +
theme(legend.position="none", 
panel.grid.minor = element_blank(), 
panel.grid.major = element_blank(), 
panel.background = element_blank())+ coord_equal()

Bimap + inset_element(leg, 0,0,0.3,0.3) 
``` 
