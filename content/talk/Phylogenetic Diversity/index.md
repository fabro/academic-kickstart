+++
title = "Phylogenetic Diversity"

# Authors. Comma separated list.
authors = ["Juliana Herrera-Perez & Fabricio Villalobos"]

  
+++

# Phylogenetic Diversity (PD) and Residual PD

**Macroecology**

Julius-Maximilians-Universität Würzburg

Phylogenetic diversity is calculated as the sum of branch lenghts of the tree that connects all species within an assemblage. 
The residuals obtain from regressing PD and Species Richness can be used as a proxy to identify important evolutionary event (speciation, extinction and dispersal) and can help us infer how have they have contributed to the patterns we observed today

Load the required packages
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

For this exercise, we'll load R data with a PAM (presence-absence matrix) and a phylogeny with species from Carnivora in South America

```         
load('exercises_data/Datacarn.RData')
```

To explore the PAM, we can plot the richness raster contained in the PresenceAbsence object

```         
pam_mamm$Richness_Raster<- terra::rast('exercises_data/richcarn.tif')
plot (pam_mamm$Richness_Raster)
```

We could something more **elegant** using ggplot

```         
SR<-as.data.frame(pam_mamm$Richness_Raster,xy=TRUE,na.rm=TRUE)%>% 
  filter (lyr.1!=0)%>%rename( "Riqueza"="lyr.1")

mapSR<-ggplot()+geom_tile(data = SR , aes(x = x, y = y, fill = Riqueza))+
  scale_fill_viridis_c(limits = c(0, 25),option = "H")+theme_void()+
  theme(axis.text.x=element_blank(),
        axis.ticks.x=element_blank(),
        axis.text.y=element_blank(),
        axis.ticks.y=element_blank())
        
mapSR
```

To calculate PD, we need the actual PAM (not the whole PresenceAbsence object), without the coordinates

```         
pa<-pam_mamm$Presence_and_Absence_Matrix
pas<-pa[,c(-1,-2)]
```

Calculate PD

```         
#calculation of PD
pdmaam<-picante::pd(pas,phymmam, include.root=FALSE)

#add a column to the PA matrix
Pd_table<-cbind(pam_mamm$Presence_and_Absence_Matrix,pdmaam)

#sort the resulting table
sort<-Pd_table%>% na.omit()%>% as.data.frame() %>% arrange(desc(SR))

#conduct a LOESS regression
model<- loess(PD ~ SR,data=sort)
```

Plot the relationship between PD and SR

```         
plot(sort$SR,sort$PD, xlab="Riqueza",ylab="PD",pch=19)

pred<-predict(model)
lines(pred, x=sort$SR, col="red")
```

What do we see?
And if we put it on a map?!

First, we can plot PD
```         
names(sort)[c(1,2)]<-c("x","y")
rPD <- residuals(model)
rPDtable<-cbind(sort,rPD)

#Map of PD 
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

Now we can plot the residuals from the PD-SR regression 
```         

positive <- rPDtable$rPD[rPDtable$rPD >= 0]
negative <- rPDtable$rPD[rPDtable$rPD < 0]

# Create 16 intervals for positive and negative residuals
breaksp <- seq(min(positive), max(positive), length.out = 16)  # 16 intervalos
breaksn <- seq(min(negative), max(negative), length.out = 16)  # 16 intervalos
breakstotal <- c(breaksn, breaksp)

# Create color palettes
colp <- viridis_pal(alpha = 1, option = "rocket", direction = -1)(20)  # Colors for positives
coln <- viridis_pal(alpha = 1, option = "mako", direction = 1)(20)     # Colors for negatives
cols <- c(coln[1:16], colp[1:16])  # Combine the palettes

# Map using ggplot
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

#Let's combined the maps 
mapSR + mapPD + maprPD
```

What do the **positive** residuals mean? and the **negative** residuals?

On one hand, positive residuals show higher PD than expected from SR. Sites with +rPD can be places where there was low and or "ancient" in situ diversification, but with several dispersal events.

On the other hand, the negative residuals show lower PD than expected from SR. Sites with -rPD indicate coexistence of lineages that were recently derived or closely related.

What if we want to look at SR and PD at the same time? We can use a bivariate map!

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
