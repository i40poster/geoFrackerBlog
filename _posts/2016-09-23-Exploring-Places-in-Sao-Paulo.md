---
layout: page
subheadline: "Exploring some services available in São Paulo"
title: "..."
teaser: "Marking some special buildings"
categories:
  - R
tags:
  - Maps
  - SAO PAULO
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetupV2)

# Let's explore data

- let's process some services available

```R
# this object contains some of the services available in GeoSampa

saopaulo.data.equipamentosList

allEquipamentos <- saopaulo.data.downloadBatch(saopaulo.data.equipamentosList )
#this creates a list of lists
collectionOfEquipamentos <-  saopaulo.data.processShapeListsAsList(allEquipamentos,"sirgas|SIRGAS")

# test your data
plot(collectionOfEquipamentos[[4]])


```

- Detect what build it is an equipament

```R
#Edification
Edificacao = geofracker.downloadAndUnzipShp(saopaulo.data.edificacaoList[79])
#http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_AGUA_RASA&arqTipo=Shapefile

Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])
plot(Edificacao1)
datasetIdx <- 25
#fix projection
proj4string(collectionOfEquipamentos[[datasetIdx]]) <- Edificacao1@proj4string



ct = rgeos::gContains( Edificacao1,collectionOfEquipamentos[[datasetIdx]],byid=TRUE, returnDense=TRUE )
#Returns a matrix of where colmns are Edificacao1
#Rows are collectionOfEquipamentos[[datasetIdx]]
ncol(ct) == length(Edificacao1)
length(ct[,1]) == length(collectionOfEquipamentos[[datasetIdx]])

Edificacao1@data$Area <- apply(ct, 2,any)
Edificacao1KnownServiceArea <- subset(Edificacao1, Edificacao1@data$Area == TRUE)


subset(Edificacao1, Edificacao1@data$Area == TRUE)

plot(Edificacao1)
#plot(Edificacao1[colsub,],add=T,col='green')
plot(collectionOfEquipamentos[[datasetIdx]],add=T,pch=16, col='red')
plot(Edificacao1KnownServiceArea,add=T,col='green')

```


Creating a function to mark special buildings:


```R

geofracker.markServiceBuildings <- function(edifications, equipamentosCollection) {
  df <- data.frame(matrix(ncol = 0, nrow = length(Edificacao1)))
  index <- 1
  i <- 0
  for (equipament in collectionOfEquipamentos){
    print(paste(length(shapeSource), "-",index))
    #fix projection
    proj4string(equipament) <- edifications@proj4string
    ct = rgeos::gContains( edifications,equipament,byid=TRUE, returnDense=TRUE )
    resultAppend <- apply(ct, 2,any)
    df[,index] <- resultAppend
    index <- index + 1
  }
  resultFinal <- apply(df,1,any)
  return(resultFinal)
}

result <- geofracker.markServiceBuildings(Edificacao1,collectionOfEquipamentos)

Edificacao1@data$SpecialArea <- result
Edificacao1KnownServiceArea <- subset(Edificacao1, Edificacao1@data$SpecialArea == TRUE)

plot(Edificacao1)
plot(Edificacao1KnownServiceArea,add=T,col='green')


```

![Sample Image]({{site.url}}/images/20160923_MarkingSpecialBuilding.png)



# References:
[1]: http://www.ats.ucla.edu/stat/r/library/advanced_function_r.htm

http://www.cookbook-r.com/Manipulating_data/Adding_and_removing_columns_from_a_data_frame/

https://cran.r-project.org/web/packages/rgeos/rgeos.pdf

https://www.r-bloggers.com/rgeos-update/

http://www.nickeubank.com/wp-content/uploads/2015/10/RGIS2_MergingSpatialData_part2_GeometricManipulations.html

https://www.nceas.ucsb.edu/scicomp/usecases/point-in-polygon

http://stackoverflow.com/questions/19002744/spover-for-point-in-polygon-analysis-in-r

http://stackoverflow.com/questions/21971447/check-if-point-is-in-spatial-object-which-consists-of-multiple-polygons-holes
