# Extracting IUCN threat data and IUCN Red List Categories using `rredlist`
## Tutorial Contents
This repository contains an R tutorial to extract species threat data from the [IUCN Threat Classification Scheme](https://www.iucnredlist.org/resources/threat-classification-scheme). It also contains the code to extract the [IUCN Red List Categories](https://www.iucnredlist.org/). 

**Note - Because threats and categories are updated through time, they may change in the future.**

This code was written by [Cerren Richards](https://github.com/CerrenRichards) and [Rob Cooke](https://github.com/03rcooke). 


## Data to download for the tutorial
We will use the `Imputed Trait Data.csv` data from Richards et al. (2021) "Biological traits of seabirds predict extinction risk and vulnerability to anthropogenic threats" that contains a list of 341 seabird species. The data can be [downloaded from Dryad](https://datadryad.org/stash/dataset/doi%253A10.5061%252Fdryad.x69p8czhd). 


- [Read the full article here.](https://onlinelibrary.wiley.com/doi/abs/10.1111/geb.13279) 
- [Read the bioRxiv preprint here.](https://www.biorxiv.org/content/10.1101/2020.09.30.321513v1) 


<img width="1393" alt="Seabirds" src="https://user-images.githubusercontent.com/39834789/191879287-6f39868f-ae66-4574-9e9b-ef8d2e4c8f03.png">

**Artwork: Cerren Richards**


## Other requirements for the tutorial
An API key is needed to download the IUCN data. APIs can be requested from the [IUCN website](https://apiv3.iucnredlist.org/api/v3/token).

## Scripts
`IUCN Tutorial.Rmd` The R Markdown file that contains the code for the tutorial.

## Contact
Any queries can be directed to **Cerren Richards** cerrenrichards@gmail.com

## IUCN Threat Classifications
**Table representing the broad IUCN threat classification categories.**

<img width="378" alt="IUCN threats" src="https://user-images.githubusercontent.com/39834789/191875364-fd627238-9f73-4dcf-8d1d-2578c0fb52b9.png">


We will use package `rredlist` to extract the IUCN data (Chamberlain, 2020). 

`rredlist` - Scott Chamberlain (2020). [rredlist: 'IUCN' Red List Client](https://CRAN.R-project.org/package=rredlist). R package version 0.7.0.

```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
# Load packages
library(rredlist); library(rlist) 
```

Load in the seabird data from Richards et al. (2021)
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
seabirds <- read.csv("Imputed Trait Data.csv")
```

We will create a vector of species names from the traits dataframe.

**IMPORTANT** - Species names must match the IUCN species names
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
spp <- as.character(seabirds$binomial) 
```

Enter your unique IUCN code here
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn_key <- "" 
```

Extract the threats for each species from th IUCN database. It creates a list. 

**NOTE** - Depending on the size of your species list, this can take a while. For 341 species, it takes 10-15 minutes to run.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn <- lapply(spp, function(x) {
  y <- rl_threats(name = x, key = iucn_key)
  Sys.sleep(2)
  # 2 second delay makes API work better - recommended by IUCN
  return(y)
})
```

Add an extra column to the dataframes in the list with the binomial names.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
for (i in 1:length(iucn)) { 
  iucn[[i]][["result"]]$binomial <- iucn[[i]][["name"]] 
}
```

Reduce the list and tidy the dataframe.

```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}

## Reduce the lists down
iucn <- Reduce(rbind, iucn) 

# remove all of the extra species names in the list
iucn[1:341] <- NULL 

# Remove all lists with species with no threats
iucn <- iucn[sapply(iucn, length)>1] 

# binds all list elements by row to make a dataframe
iucn <- list.rbind(iucn)

# Delete the columns that will not be used in further analyses
iucn <- iucn[- c(3:7)] 

```


## Subset species at risk to specific threats

If you are interested in extracting only a subset of threats, you can use the code below.

Here we subset all the seabird species that are known to be threatened from oil drilling, shipping lanes and subsistence fishing. 

We must first check the unique [IUCN Codes](https://www.iucnredlist.org/resources/threat-classification-scheme):

-	3.1 Oil & gas drilling
-	4.3 Shipping lanes
-	5.4.1 Intentional use: subsistence/small scale (species being assessed is the target)[harvest]
-	5.4.3 Unintentional effects: subsistence/small scale (species being assessed is not the target)[harvest]


```{r, eval = FALSE}

# Subset based on the IUCN codes
Threats <- iucn[iucn$code %in% c("3.1", # Oil & gas drilling
                                 "4.3", # Shipping lanes
                                 "5.4.1", # Intentional subsistence fishing
                                 "5.4.3"), ] # Unintentional subsistence fishing

```



## IUCN Red List Categories

<img width="1118" alt="Categories" src="https://user-images.githubusercontent.com/39834789/191875626-f479e8a9-d6f3-4505-bcb5-812e6fa5512f.png">

The following code follows a similar approach as seen above to extract the IUCN Red List Categories for the seabirds in Richards et al. (2021). 


Extract the IUCN categories for each species from the IUCN database. 
**NOTE** - this code can take ~15 minutes to run.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- lapply(spp, function(x) {
  y <- rl_history(name = x, key = iucn_key)
  Sys.sleep(2)
  # 2 second delay makes API work better - recommended by IUCN
  return(y)
})
```

Add an extra column to the dataframes in the list with the binomial names:
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
for (i in 1:length(iucn.red)) { 
  iucn.red[[i]][["result"]]$binomial <- iucn.red[[i]][["name"]] 
}
```

Thayer's Gull is Not Evaluated by the IUCN, so we will remove it.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- iucn.red[sapply(iucn.red, length)>1] 
```

Binds all list elements by row to make a dataframe.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- list.rbind(iucn.red) 
```

Remove all of the extra species names in the list.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red[1:341] <- NULL 
```

Deletes Thayer's Gull.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red[[134]] <- NULL 
```

Binds all list elements by row to make a dataframe.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- list.rbind(iucn.red) 
```

Remove all the historic information and only keep the most recent IUCN classification. 
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- iucn.red %>% distinct(binomial, .keep_all = TRUE)
```

Delete the columns that will not be used further.
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
iucn.red <- iucn.red[- c(1,3)] 
```

Rename the code column to IUCN
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
seabirds <- left_join(seabirds, iucn.red, by = "binomial")
```

# rename the column
```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
colnames(seabirds)[colnames(seabirds) == 'code'] <- 'IUCN'
```
