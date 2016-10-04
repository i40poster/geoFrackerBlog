---
layout: page
subheadline: "Exploring the location of São Paulo services"
title: "Let's play with voronoi and São Paulo data"
teaser: "Voronoi, and some São Paulo data."
categories:
  - R
tags:
  - Maps
  - SAO PAULO
  - Voronoi

comments: true
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetupV2)

# New function

```
geofracker.voronoipolygons = function(rawLayer)

```

# Let's explore data

- Define voronois for all the "utilities":

```R
Abastecimento = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=03_Equipamentos%5C%5CAbastecimento%5C%5CShapefile%5C%5CEQUIPAMENTOS_SHP_TEMA_ABASTECIMENTO&arqTipo=Shapefile")

Abastecimento1 <- readOGR(dsn=Abastecimento$dir[1], layer=Abastecimento$shapeclass[1])

data = geofracker.voronoipolygons(Abastecimento1)
proj4string(data) <- Abastecimento1@proj4string
plot(data)
#add points
points(Abastecimento1)



```

Further Draws:

```R

spplot(data, "dummy")
```

- Get the centroid of a polygon from an edification:

```R

Edificacao = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_SE&arqTipo=Shapefile")
Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])
plot(Edificacao1)
#centers of edifications
pointsEdificacao <- rgeos::gCentroid(Edificacao1, byid=TRUE)
proj4string(pointsEdificacao) <- Edificacao1@proj4string

```
> Generate a list of "Edification" center of mass.

- Use the voronoi to define which  a edification polygon belongs to a voronoi polygon.

> Exploring "over" to check which polygon the building belongs


```R
#Get region
pointsEdificacao$voronoiData <- over(pointsEdificacao, data)$voronoi.zone
proj4string(data)<-Edificacao1@proj4string
identical(proj4string(data), proj4string(Edificacao1))
Edificacao1@proj4string

pointsEdificacao$voronoiData <- over(pointsEdificacao,as(data,"SpatialPolygons"))
plot(data, border = "darkgrey")
points(pointsEdificacao, col = "red", pch = 16)




```

- Calculate the distance of to the closest Voronoi center

```R
pointsEdificacao$distanceUtil <- pointDistance(pointsEdificacao,coordinates(data[na.omit(as.character(pointsEdificacao$voronoiData)), ]))

mapdata <- data.frame(pointsEdificacao)

```

- Rendering the results:

```R

ggplot(data= mapdata, aes(x=x, y=y, color=distanceUtil)) +  geom_point( ) + scale_colour_gradient(limits=c(3, 1000), low="green", high="red") + coord_fixed()

```

> - The most red in the color means more distance from the equipament in study, the greenish it is closer it is.
> - The points are the center of the equipament center of mass.

- Create a new column in the data to have the distance from given "utilities zone" to the closest voronoi centroid
- use this distance to a compose a score.


# Extra Results:

## Max distance to a equipament in study

```R
max(pointDistance(pointsEdificacao,coordinates(data[na.omit(as.character(pointsEdificacao$voronoiData)), ])))
```

## Coordinates of the centroids

```R

data[na.omit(as.character(pointsEdificacao$voronoiData)), ]
#centroid of the region a point belongs

coordinates(data[na.omit(as.character(pointsEdificacao$voronoiData)), ])
```




## Bonus commands:
Removing duplicated objects, to clean data:

```R
zd <- zerodist(Abastecimento1)
cleanedData <- subset(Abastecimento1, !(1:nrow(Abastecimento1) %in% zd[,2]))
length(cleanedData)
```


## Exploring unique names and links:

```R
head(Abastecimento1)
# eq_id - candidate id for equipamen (need to check it is unique)

head(Edificacao1)
# ed_id - candidate id for edification (need to check it is unique)
```

# References:


- further reference:
 <http://stackoverflow.com/questions/24236698/voronoi-diagram-polygons-enclosed-in-geographic-borders>

- to combine voronoy with a map, for more precise result:

 <http://stackoverflow.com/questions/12156475/combine-voronoi-polygons-and-maps/12159863#12159863>
