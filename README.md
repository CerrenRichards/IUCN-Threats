# IUCN-Threats
## Tutorial Contents
This repository contains an R tutorial to extract species threat data from the IUCN Threat Classification Scheme: https://www.iucnredlist.org/resources/threat-classification-scheme. It also contains the code to extract the IUCN Red List Categories: https://www.iucnredlist.org/. 
Note - Because threats and categories are updated through time, they may change in the future.


## Data to download for the tutorial
We will use the list of seabirds downloaded from Richards et al. 2021: https://datadryad.org/stash/dataset/doi%253A10.5061%252Fdryad.x69p8czhd 

## Other requirements for the tutorial
An API key is needed to download the IUCN data. APIs can be requested from: https://apiv3.iucnredlist.org/api/v3/token



## IUCN Threat Classifications

<img width="378" alt="IUCN threats" src="https://user-images.githubusercontent.com/39834789/191875364-fd627238-9f73-4dcf-8d1d-2578c0fb52b9.png">
Table representing the broad IUCN threat classification categories.


```{r message=FALSE, error=FALSE, warning=FALSE, eval = FALSE}

# Load packages
library(rredlist); library(rlist) 

# Load the names of seabirds downloaded from Richards et al. (2021)
seabirds <- read.csv("Imputed Trait Data.csv")

# Create a vector of species names from the traits dataframe
# IMPORTANT - Species names must match the IUCN species names
spp <- as.character(seabirds$binomial) 

# Enter your unique IUCN code here
# Request from: https://apiv3.iucnredlist.org/api/v3/token 
iucn_key <- "" 

# Extract the threats for each species from th IUCN database.
# Creates a list
# NOTE - Depending on the size of your species list, this can take a while
# For 341 species, it takes 10-15 minutes to run
iucn <- lapply(spp, function(x) {
  y <- rl_threats(name = x, key = iucn_key)
  Sys.sleep(2)
  # 2 second delay makes API work better - recommended by IUCN
  return(y)
})

### Add an extra column to the dataframes in the list with the binomial names:
for (i in 1:length(iucn)) { 
  iucn[[i]][["result"]]$binomial <- iucn[[i]][["name"]] 
}

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




## IUCN Red List Categories

 
 
<img width="1118" alt="Categories" src="https://user-images.githubusercontent.com/39834789/191875626-f479e8a9-d6f3-4505-bcb5-812e6fa5512f.png">

## Scripts
`IUCN Tutorial.Rmd` The R Markdown file that contains the code for the tutorial.

## Contact
Any queries can be directed to **Cerren Richards** cerrenrichards@gmail.com
