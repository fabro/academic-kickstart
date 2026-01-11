+++
title = "Null Models"

# Authors. Comma separated list.
authors = ["Fabricio Villalobos"]

  
+++

# Null models for the assembly of species assemblages

**Macroecology**

Julius-Maximilians-Universität Würzburg


#### What are the questions? 
##### 1) Observed species assemblages are different from random samples of species? "full null"
##### 2) Observed species assemblages are different from random but restricted samples of species? "contrained null" (biological information)


Create the data
```{r eval=FALSE}
#vector with the number and identity of genera present in the species pool
generos <- as.vector(c("a","b","c","d","e","f"))

#vector with observed species richness in each island 
richness_coms <- as.vector(c(13,8,17,6,2))

#vector with number of genera in each island (in the same order as the richness)
genrs_coms <- as.vector(c(3,6,2,5,1))
```

### STEP 1 of null models: Calculate the desired index of assemblage structure
```{r eval=FALSE}
# create a vector with the observed S/G ratios for each island/assemblage
s_g_obs <- as.vector(richness_coms/genrs_coms)
# calculate the mean value of S/G ratio across all islands/assemblages
mean_sgo <- mean(s_g_obs)
```

### STEP 2. Create the null/random assemblages
```{r eval=FALSE}
# one random assemblage with the observed richness of the first assemblage (e.g. richness_coms[1])
c1 <- sample(generos, richness_coms[1],replace=T)
# another random assemblage with the richness of the second assemblage (e.g. richness_coms[2])
c2 <- sample(generos, richness_coms[2],replace=T)
#calculate the S/G ratio for the first random assemblage
sg_null1 <- length(unique(c1))/genrs_coms[1]
```

#### and I have to do that for each one? what a hustle! Let's try something simpler:

Create objects to save results 
```{r eval=FALSE}
# create a matrix that will be filled with the data from random assemblages
com <- matrix(0, nrow=max(richness_coms), ncol=length(richness_coms))
# create a matrix to save the calculated S/G ratios of the random assemblages
sg_nulls <- matrix(0, nrow=length(richness_coms), ncol=1)
```

### STEP 3. Calculate the indices for each assemblage
```{r eval=FALSE}
# how do we save the results?
# we used the vectors created above

# create a matrix that will be filled with the data from random assemblages
coms <- matrix(0, nrow=max(richness_coms), ncol=length(richness_coms))
# create a matrix to save the calculated S/G ratios of the random assemblages
sg_nulls <- matrix(0, nrow=length(richness_coms), ncol=1)

# and we fill them up!

sg_nulls[1,] <- sg_null1
coms[1:richness_coms[1],1] <- c1
mean(sg_nulls)
```

#### and now? one by one? NO! let's automate and save effort!!
we use "loops" in R to repeat all the steps

```{r eval=FALSE}
# create null assemblages with random sampling from the species pool
for (i in 1:length(richness_coms)){
	
		coms[1:richness_coms[i],i] <- sample(generos, richness_coms[i], replace=T)
	
}

# calculate S/G ratios for each null assemblage and save them 
for (i in 1:ncol(coms)){
	
	sg_nulls[i,1]	<- richness_coms[i]/length(unique(coms[1:richness_coms[i],i]))	
	
}
```

BUT, this only generates ONE null scenario, we much more!!!
Then, we can built our own functions: one to built one scenario and a second one to repeat it many times  

```{r eval=FALSE}
null_sg <- function(generos, richness_coms){
	# create matrices to save the results
	coms <- matrix(0, nrow=max(richness_coms), ncol=length(richness_coms))
	sg_nulls <- matrix(0, nrow=length(richness_coms), ncol=1)
	
		# create the null assemblages 
		for (i in 1:length(richness_coms)){
			
				coms[1:richness_coms[i],i] <- sample(generos, richness_coms[i], replace=T)
			
		}
	
	
		# calculate S/G ratios for each null assemblage and save them 
		for (i in 1:ncol(coms)){
			
			sg_nulls[i,1]	<- richness_coms[i]/length(unique(coms[1:richness_coms[i],i]))	
			
		}
	
		mean(sg_nulls)
}
```

Now, the second function, to repeat the previous one and plot the results against the observed S/G value

```{r eval=FALSE}
nulls_sg_sim <- function(generos, richness_coms, s_g_obs, sims){

	sg_means <- matrix(0, nrow=sims, ncol=1)
	
	for (i in 1:sims){
		
		sg_means[i,1] <- null_sg(generos, richness_coms)
	}
	
		if (mean(s_g_obs) > max(sg_means)){
			
			hist(sg_means, xlim=c(0,mean(s_g_obs)))
			abline(v=mean(s_g_obs),col="red")
			
		} else {
			
			hist(sg_means, xlim=c(0,max(sg_means)))
			abline(v=mean(s_g_obs), col="red")
		}
	
	
}
```

## Constrained null model (with additional biological information)
Constrained null model: first, create a vector with the probabilities for each genera (keeping their order), based on their own observed richness. 
This means that the chance of sampling a particular genus depends on its richness: the richer, the higher the chance of being sampled 

Create the vector of probabilities
```{r eval=FALSE}
prob_sp <- as.vector(c(1/18,2/18,2/18,3/18,4/18,6/18))
```

Change the functions to consider the probabilities of each genus

```{r eval=FALSE}
nullcons_sg <- function(generos, richness_coms, prob_sp){
	coms <- matrix(0, nrow=max(richness_coms), ncol=length(richness_coms))
	sg_nulls <- matrix(0, nrow=length(richness_coms), ncol=1)
	
		for (i in 1:length(richness_coms)){
			
				coms[1:richness_coms[i],i] <- sample(generos, richness_coms[i], replace=T, prob_sp)
			
		}
		
		for (i in 1:ncol(coms)){
			
			sg_nulls[i,1]	<- richness_coms[i]/length(unique(coms[1:richness_coms[i],i]))	
			
		}
	
		
		mean(sg_nulls)
} 
```

```{r eval=FALSE}
nullcons_sg_sim <- function(generos, richness_coms, s_g_obs, prob_sp, sims){
	
	sg_means <- matrix(0, nrow=sims, ncol=1)
	
	for (i in 1:sims){
		
		sg_means[i,1] <- nullcons_sg(generos, richness_coms, prob_sp)
	}
	
		if (mean(s_g_obs) > max(sg_means)){
			
			hist(sg_means, xlim=c(0,(mean(s_g_obs))))
			abline(v=mean(s_g_obs), col="red")
			
		} else {
			hist(sg_means, xlim=c(0,max(sg_means)))
			abline(v=mean(s_g_obs),col="red")
		}
	
	
}
```
####**NOTE**: We have the cases for each individual assemblage, not for the mean S/G. You could explore/create those cases
### GOOD LUCK!!

