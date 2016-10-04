---
layout: page
#
# Content
#
subheadline: "Workbench Setup"
title: "Initial Setup for your RStudio"
teaser: "Common tool set to explore RStudio for playing with maps and data"
categories:
  - R
tags:
  - Maps
  - SAO PAULO
  - Docker
  - Workbench
#
# Styling
#
image:
  #header: ""
  #thumb: "you-can-delete-me-thumb.png"
  #homepage: "you-can-delete-me-homepage.png"
  #caption: "Caption?"
  #url: "http://phlow.de/"

permalink: /RStudioSetup

comments: true
---

# Pre-Requirements

The next steps will require you to have docker.

## what is docker?
<div class="flex-video">
  <iframe src="//www.youtube.com/embed/aLipr7tTuA4" frameborder="0"></iframe>
</div>


For details on how to install and setup docker check out this page:
https://docs.docker.com/

For Windows and Mac check out:
https://www.docker.com/products/docker-toolbox



# Running your RStudio


Let's create a RStudio using Docker:

```bash
docker run -d -p 8790:8787 --name r_workbench it4poster/geofracker

##Open your browser at your docker host IP at port 8790
# Ususally for mac or windows: http://192.168.99.100:8790
# if you are using docker machine
# For linux user, usually: http://<your ip>:8790
# or: http://127.0.0.1:8790

```
At you your browser you will see a login page, for login use:
User: rstudio
Password: rstudio

Now execute the following to install some of the libs we will be using on our samples trough the blog:

```R
#!/usr/bin/env Rscript
install.packages("rgeos")
install.packages("rgdal")
# raster install not working
install.packages('raster',  repos='http://cran.us.r-project.org')
install.packages('ncdf4')
install.packages("xlsx")
install.packages("ggmap")
library(raster)
library(rgdal)

# installing dowload helper
## 1. install
install.packages("devtools")
library(devtools)
devtools::install_github("i40poster/geoSampaRHelper")

# using dowload helper
## 2 .Sample of use:
library(helper4geosampa)

# To conver GeoJSON
install.packages("geojsonio")
library("geojsonio")


# helper function to convert maps at the south hemisphere:
utm2decimalSouth <- function(data,zone,datum){
    #coordinates(newData) <- c("easting","northing")
    crs <- paste0("+proj=utm+zone=",zone,"+datum=",datum)
    data@proj4string@projargs <- paste0("+proj=utm"," +south  +zone=",zone," +datum=",datum)
    spTransform(data, CRS("+proj=longlat"))}
```

Don't worry **rgdal** has some OS dependencies, but your linux docker image has already it installeed for you.


For testing it just check out some of our posts in the blog.
