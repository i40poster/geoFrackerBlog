---
layout: page
title: "Calculateing magic distances, based from key equipaments"
teaser: "Voronoi, Edification, and tables."
subheadline: "..."
categories:
  - R
tags:
  - Maps
  - SAO PAULO


comments: true
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetupV2)

# Let's explore data

- First Reading some data:

```R
#paste(paste("col", 1:3, sep = ""),paste("col", 1:3, sep = ""))


# Equipament:
allEquipamentos <- saopaulo.data.downloadBatch(saopaulo.data.equipamentosList )
#this creates a list of lists
collectionOfEquipamentos <-  saopaulo.data.processShapeListsAsList(allEquipamentos,"sirgas|SIRGAS")

Equipament1 <- collectionOfEquipamentos[[1]]

data = geofracker.voronoipolygons(Equipament1)
proj4string(data) <- Equipament1@proj4string


# test your data
plot(Equipament1)

# Edification:
Edificacao = geofracker.downloadAndUnzipShp(saopaulo.data.edificacaoList[79])

Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])


#centers of edifications
pointsEdificacao <- rgeos::gCentroid(Edificacao1, byid=TRUE)

#Calibrate Zones
proj4string(pointsEdificacao) <- Edificacao1@proj4string
proj4string(data) <- Edificacao1@proj4string
proj4string(Equipament1) <- Edificacao1@proj4string

#Get region
pointsEdificacao$voronoiData <- over(pointsEdificacao, data)$voronoi.zone
proj4string(data)<-Edificacao1@proj4string
identical(proj4string(data), proj4string(Edificacao1))
Edificacao1@proj4string

pointsEdificacao$voronoiData <- over(pointsEdificacao,as(data,"SpatialPolygons"))
plot(data, border = "darkgrey")
points(pointsEdificacao, col = "red", pch = 16)

pointsEdificacao$distanceUtil <- pointDistance(pointsEdificacao,coordinates(data[na.omit(as.character(pointsEdificacao$voronoiData)), ]))

mapdata <- data.frame(pointsEdificacao)



```

- Rendering the results:

```R

ggplot(data= mapdata, aes(x=x, y=y, color=distanceUtil)) +  geom_point( ) + scale_colour_gradient(limits=c(3, 1000), low="green", high="red") + coord_fixed()

```

# Exploring

```R
eq_id

for (sourcePath in collectionOfEquipamentos){
  print(names(sourcePath))
}

saopaulo.data.discoveryShapedetails <- function(listOfShape) {
  i <- 1
  for (sourcePath in listOfShape){
    #print(sourcePath)
    #print(sourcePath@data$eq_id)
    #print(names(sourcePath))
    #print(class(sourcePath)[1])
    #print(names(collectionOfEquipamentos[[1]]))
    #print(length(names(sourcePath)))
    print(paste("i:",i))
    #print(paste("head:",head(sourcePath$eq_classe)))
    #print(paste("nm_tema_eq:",!is.null(sourcePath$nm_tema_eq)))
    #if(!is.null(sourcePath$nm_classe_))
    #  print(paste("nm_classe_:",sourcePath$nm_classe_))
    #else if(!is.null(sourcePath$eq_classe))
    #  print(paste("eq_classe:",sourcePath$eq_classe))


    if(!is.null(sourcePath$nm_classe_))
      print("OK")
    else if(!is.null(sourcePath$eq_classe))
      print("OK")
    else if(!is.null(sourcePath$source_filename))
         print(paste("source_filename:",sourcePath$source_filename))
         #print("OK")
    else
      print(names(sourcePath))
    i <- i +1
  }
}

saopaulo.data.discoveryTypes<- function(input) {

  i <- 1
  for (sourcePath in input){
    print(paste("i:",i))
    print(unique(sourcePath$source_filename))

    if(!is.null(sourcePath$nm_classe_))
      print(unique(sourcePath$nm_classe_))
    else if(!is.null(sourcePath$eq_classe))
      print(unique(sourcePath$eq_classe))
    print(head(coordinates(sourcePath)))
    i <- i +1
  }
}
saopaulo.data.discoveryTypes(collectionOfEquipamentos)



saopaulo.data.getClass <- function(input) {
  i <- 1
  result <- data.frame(matrix(vector(), length(collectionOfEquipamentos), 2,
                  dimnames=list(c(), c("file", "class"))),
                  stringsAsFactors=F)
  mapCollection <- Map(class,collectionOfEquipamentos)
  for (sourcePath in input){
    print(result[,1])
    result[,1][i]<- i
    result[,2][i]<- mapCollection[[i]][1]
    i <- i + 1
  }
  return(result)

}


saopaulo.data.intersectList <- function(input) {
  i <- 1
  result <- intersect(names(input[[i]]),names(input[[i]]))
  for (sourcePath in input){
    if (i != 1 ){
      result <- intersect(result,names(input[[i]]))
    }
    i <- i + 1
  }
  return(result)

}
saopaulo.data.intersectList(collectionOfEquipamentos)
#no intersection on the data available

saopaulo.data.getClass(collectionOfEquipamentos)

Reduce(intersect, Map(names,collectionOfEquipamentos))
```

# D3.JS Rendering Section
Add the SVG code here

# References:

geofracker.removeServiceBuildings

#geofracker.utm2decimalSouth

#geofracker.utm2decimalNorth
