# IUCN-Threats
## Tutorial Contents
This repository contains an R tutorial to extract species threat data from the IUCN Threat Classification Scheme: https://www.iucnredlist.org/resources/threat-classification-scheme. It also contains the code to extract the IUCN Red List Categories: https://www.iucnredlist.org/. 

**Note - Because threats and categories are updated through time, they may change in the future.**


## Data to download for the tutorial
We will use the list of seabirds downloaded from Richards et al. 2021: https://datadryad.org/stash/dataset/doi%253A10.5061%252Fdryad.x69p8czhd 

## Other requirements for the tutorial
An API key is needed to download the IUCN data. APIs can be requested from: https://apiv3.iucnredlist.org/api/v3/token

## Scripts
`IUCN Tutorial.Rmd` The R Markdown file that contains the code for the tutorial.

## Contact
Any queries can be directed to **Cerren Richards** cerrenrichards@gmail.com

## IUCN Threat Classifications
**Table representing the broad IUCN threat classification categories.**

<img width="378" alt="IUCN threats" src="https://user-images.githubusercontent.com/39834789/191875364-fd627238-9f73-4dcf-8d1d-2578c0fb52b9.png">


We will use package `rredlist` to extract the IUCN data. Here we also load in the seabird data.

```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}
# Load packages
library(rredlist); library(rlist) 

# Load the names of seabirds downloaded from Richards et al. (2021)
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

We must first check the unique IUCN Codes = https://www.iucnredlist.org/resources/threat-classification-scheme

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

```{r}
# Extract the IUCN categories for each species from the IUCN database. 
# NOTE - this code can take ~15 minutes to run.
iucn.red <- lapply(spp, function(x) {
  y <- rl_history(name = x, key = iucn_key)
  Sys.sleep(2)
  # 2 second delay makes API work better - recommended by IUCN
  return(y)
})

### Add an extra column to the dataframes in the list with the binomial names:
for (i in 1:length(iucn.red)) { 
  iucn.red[[i]][["result"]]$binomial <- iucn.red[[i]][["name"]] 
}

# Thayer's Gull is Not Evaluated by the IUCN, so we will remove it
iucn.red <- iucn.red[sapply(iucn.red, length)>1] 

# binds all list elements by row to make a dataframe
iucn.red <- list.rbind(iucn.red) 

# remove all of the extra species names in the list
iucn.red[1:341] <- NULL 

# deletes Thayer's Gull
iucn.red[[134]] <- NULL 

# binds all list elements by row to make a dataframe
iucn.red <- list.rbind(iucn.red) 

# remove all the historic information and only keep the most recent IUCN classification 
iucn.red <- iucn.red %>% distinct(binomial, .keep_all = TRUE)

# Delete the columns that will not be used in further analyses
iucn.red <- iucn.red[- c(1,3)] 

# Rename the code column to IUCN
traits <- left_join(seabirds, iucn.red, by = "binomial")

# rename the column
colnames(seabirds)[colnames(seabirds) == 'code'] <- 'IUCN'
```


