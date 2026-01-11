+++
title = "Ecogeographic Rules"

# Authors. Comma separated list.
authors = ["Fabricio Villalobos"]

  
+++

# Ecogeographic rules: the case of Rapoport

**Macroecology**

Julius-Maximilians-Universität Würzburg

## Rapoport's Rule

#### Get the trait of interest: geographic range size 

Load packages
```{r eval=FALSE}
library(letsR)
```

Load observed PAM (from the bat data)
```{r eval=FALSE}
load("exercises_data/bats_PAM.Rdata")
bats.pam <- m.PAM
```

Calculate the geographic range sizes of species
```{r eval=FALSE}
bats.ranges <- lets.rangesize(bats.pam, units = "cell")
```
Now we have a vector with range sizes (number of occupied grid cells) of all species


#### Interspecific approach
This approach is based on the analysis of species as units of observation, then we need characterize their properties. 
Besides the trait (range size), we need their location along the gradient (latitude).

Get the location of each species. In this case, their 'midpoint'
```{r eval=FALSE}
bats.midpoints  <-lets.midpoint(bats.pam, planar = F)
```

Done! We have the data, now test the latitude~range size relationship
```{r eval=FALSE}
bats.lm <- lm(bats.ranges[,1]~abs(bats.midpoints[,3]))

summary(bats.lm)
```

But, what about the evolutionary aspect?
Let's evaluate it!
```{r eval=FALSE}
#Load packages for evolutionary data/methods
library(ape)
library(caper)
#Load the phylogeny of bats
bats.tree <- read.tree("exercises_data/bats_tree")

bats.tree$tip.label <- gsub("_"," ",bats.tree$tip.label)

#prepare the data
bats.data  <- cbind(bats.midpoints[,c(1,3)],bats.ranges)
colnames(bats.data) <- c("Species","MidLat","Range_size")
#create a "comparative data" object needed for the analysis, according to the package being used
bats.compdata <- comparative.data(bats.tree,bats.data,Species,vcv=T)

#fit the phylogenetic regression model (PGLS: phylogenetic generalized least squares)
bats.pgls <- pgls(Range_size~abs(MidLat),bats.compdata,lambda='ML')

summary(bats.pgls)
```
So? What results do we have?


#### Assemblage-based approach
This approach is based in evaluating sites/assemblages (grid cells) as units of observation, then we need to characterize their properties. 
We need to calculate a metric for each assemblage/cell, such as the mean trait value for the species set as well as the location of the cell (latitude).

```{r eval=FALSE}
#Get the mean values per assemblage
#for that, we have letsR! (shameless selfpromotion, again ;) )
bats.medianRS <- lets.maplizer(bats.pam,bats.ranges[,1],rownames(bats.ranges), fun = median, ras=T)

#the location of each cell/assemblage can be obtain directly from the PAM that we already have (the first two columns).
#Then, we can simply take the latitude and test the desired relationship between the median trait (range size) and latitude
bats.lmSites  <- lm(bats.medianRS$Matrix[,3]~abs(bats.medianRS$Matrix[,2]))

summary(bats.lmSites)
```
So? What results do we have?

Now, include the evolutionary aspect based on the PVR approach (Phylogenetic eigenVector Regression; Diniz-Filho et al. 1998) to estimate the phlogenetic (P) and specific (S) components
```{r eval=FALSE}
#Obtain the P and S components. But, how? We have letsR! Unfortunately, no, the function is not in the package yet but you have it in your data folder
#load additional packages
library(picante)
library(vegan)
#load the function ( .R file)
source("exercises_data/p_s_components.R")

#run the function
bats.pscomps <- p.s.comps(phy = bats.tree,sppnames = bats.midpoints[,1],data = bats.ranges,colnum = 1)

#spatialize the components, with letsR of course ;) 
# P component
bats.meanP <- lets.maplizer(bats.pam,bats.pscomps[[2]][,1],bats.midpoints[,1], fun = mean, ras=T)
# S component
bats.meanS <- lets.maplizer(bats.pam,bats.pscomps[[2]][,2],bats.midpoints[,1], fun = mean, ras=T)

#lineal models
# Phylogenetic component
bats.lm.meanP <- lm(bats.meanP$Matrix[,3]~abs(bats.pam$P[,2]))

summary(bats.lm.meanP)

# Specific component
bats.lm.meanS <- lm(bats.meanS$Matrix[,3]~abs(bats.pam$P[,2]))

summary(bats.lm.meanS)
```

Lets plot the components in the map!
```{r eval=FALSE}
par(mfrow=c(1,3))
#original trait
plot(bats.medianRS$Raster, main="observed range size")
#phylogenetic component
plot(bats.meanP$Raster, main="phylogenetic component")
#specific component
plot(bats.meanS$Raster, main= "specific component")

```

and now?
What can we conclude?
Do the patterns are maintained? Y/N? Why?
