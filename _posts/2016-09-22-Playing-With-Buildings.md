---
layout: page
subheadline: "Exploring the location of São Paulo services"
title: "Let's play with São Paulo Edifications center of mass"
teaser: "2d Buildings Data"
categories:
  - R
tags:
  - Maps
  - SAO PAULO
  - Edifications
  - Buildings
  - Polygons
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetup)

# Let's explore some data

Running your workspace: followw the steps described [here](https://hub.docker.com/r/it4poster/geofracker/)

```bash
docker pull it4poster/geofracker
docker run -d -p 8787:8787 -p 6311:6311 it4poster/geofracker
```

Open your browser at http://dockerip:8787/

Login:

| user: | rstudio |
| password: | rstudio|


At your browser rstudio:

```R
source('/home/rstudio/setupLibs.R')
#wait the installs to be completed
```

# Playing with the data
Exploring a basic São Paulo Data:

* geofracker.downloadAndUnzipShp(URL) -> Allows you to download a shapefile for processing.

```R

Edificacao = geofracker.downloadAndUnzipShp(saopaulo.data.edificacaoList[1])
#http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_AGUA_RASA&arqTipo=Shapefile

Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])
plot(Edificacao1)
#centers of edifications
pointsEdificacao <- rgeos::gCentroid(Edificacao1, byid=TRUE)
proj4string(pointsEdificacao) <- Edificacao1@proj4string
plot(pointsEdificacao)


```

```
 saopaulo.data.edificacaoList[1] = http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_AGUA_RASA&arqTipo=Shapefile
```

> **saopaulo.data.edificacaoList**: It is list of files available already embeed on this studio version to help you to explore the data. e.g: saopaulo.data.edificacaoList[1] is the "5CSHP_edificacao_AGUA_RASA" file.

# Exploring the Center of Mass

```R
pointsEdificacao <- rgeos::gCentroid(Edificacao1, byid=TRUE)

proj4string(pointsEdificacao) <- Edificacao1@proj4string

plot(pointsEdificacao)
```


# Demo

<iframe width='970' height='546'
src="https://www.youtube.com/embed/kz_DIhT8kZc">
</iframe>

# References:
