---
layout: page
#
# Content
#
subheadline: "Sample Map Rendering"
title: "My first Map"
teaser: "Full sample of starting from scratch up to a GeoJSON file"
categories:
  - R
  - GeoJSON
tags:
  - Maps
  - SAO PAULO
#
# Styling
#
image:
  #header: ""
  #thumb: "you-can-delete-me-thumb.png"
  #homepage: "you-can-delete-me-homepage.png"
  #caption: "Caption?"
  #url: "http://phlow.de/"
---
# Test mix HTML and HTML

## Running your RStudio
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

Now execute the following:

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

```
# Exploring a sample data:
Source: http://geosampa.prefeitura.sp.gov.br/

```R
Abastecimento = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=03_Equipamentos%5C%5CAbastecimento%5C%5CShapefile%5C%5CEQUIPAMENTOS_SHP_TEMA_ABASTECIMENTO&arqTipo=Shapefile")

Abastecimento1 <- readOGR(dsn=Abastecimento$dir[1], layer=Abastecimento$shapeclass[1])

#function to convert UTM to Degrees




geojson_write( us.cities[1:2, ], lat = 'lat', lon = 'long',file = "/home/rstudio/sample.json")

utm2decimalSouth <- function(data,zone,datum){
    #coordinates(newData) <- c("easting","northing")
    crs <- paste0("+proj=utm+zone=",zone,"+datum=",datum)
    data@proj4string@projargs <- paste0("+proj=utm"," +south  +zone=",zone," +datum=",datum)
    spTransform(data, CRS("+proj=longlat"))}


#Converting the Coordinates from UTM to Degrees
Abastecimento1inDegrees <- utm2decimalSouth(Abastecimento1,23,"WGS84")
head(coordinates(Abastecimento1inDegrees))
```

Expected Result:
```text
coords.x1 coords.x2
[1,] -46.62910 -23.55624
[2,] -46.61584 -23.53899
[3,] -46.62229 -23.57894
[4,] -46.70011 -23.52669
[5,] -46.60064 -23.55279
[6,] -46.59940 -23.54647
```

## Exporting
```R
Abastecimento1iDF$lat <- coordinates(Abastecimento1inDegrees)[,2])
Abastecimento1iDF$long <- coordinates(Abastecimento1inDegrees)[,1])
Abastecimento1iDF$variable <- Abastecimento1inDegrees$eq_nome
Abastecimento1iDF <- data.frame(Abastecimento1iDF_Lat, Abastecimento1iDF_Long, Abastecimento1iDF$variable )
geojson_write(Abastecimento1iDF, lat = 'Abastecimento1iDF_Lat', lon = 'Abastecimento1iDF_Long',file = "/home/rstudio/Abastecimento1")
```

File Saved at: /home/rstudio/Abastecimento1
Copying it to your machine:
```bash
docker cp r_workbench:/home/rstudio/Abastecimento1.geojson ~/Downloads/
```

You can test the generated file at: http://geojson.io/
20160821.SampleGeoJSON.png
![GEOJSON Sample]({{site.url}}/images/20160821.SampleGeoJSON.png)


Reference: https://cran.r-project.org/web/packages/geojsonio/README.html

# D3.JS Rendering Section

<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://d3js.org/d3-collection.v1.min.js"></script>
<script src="https://d3js.org/d3-dispatch.v1.min.js"></script>
<script src="https://d3js.org/d3-dsv.v1.min.js"></script>
<script src="https://d3js.org/d3-request.v1.min.js"></script>
<style> /* set the CSS */
#viz {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
}
</style>

<div id="viz"></div>
<script>

/* code reused from the following stackoverflow question:
              http://stackoverflow.com/questions/14492284/center-a-map-in-d3-given-a-geojson-object */
    var width = 900,
        height = 900;
    console.log("{{site.url}}/articlesData/Abastecimento1.geojson");
    var svg = d3.select("#viz").append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("class", "svg");

    // load geojson and do stuff in a callback function...
    //Fixed projection to be closer to what we see on GeoSampa
    console.log("{{site.url}}/articlesData/Abastecimento1.geojson");

    //https://raw.githubusercontent.com/alignedleft/d3-book/master/chapter_12/


    //this not works on github pages.. not sure why yet
    d3.json("{{site.url}}/articlesData/Abastecimento1.geojson",
    d3.json("https://raw.githubusercontent.com/i40poster/geoFrackerBlog/master/articlesData/Abastecimento1.geojson",
    function(error, data){
        // console.log the data
        console.log(data);

        // create a unit projection
        var projection = d3.geo.mercator()
            .scale(1)
            .translate([0,0]);

        // create a path generator.
        console.log( d3.geo.path());
        var path = d3.geo.path()
            .projection(projection)
            .pointRadius(function(d) {
              console.log(d);
              return 2;
            //  return d.properties.mag;
            });

        // compute bounds of a point of interest, then derive scale and translate
        var b = path.bounds(data),
            s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height),
            t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2];

        // update the projection to use computed scale and translate....
        projection
            .scale(s)
            .translate(t);


        // calculate and draw a bounding box for the geojson
        svg.append("rect")
            .attr('width', width)
            .attr('height', height)
            .style('stroke', 'black')
            .style('fill', '#dfd');

        // draw the svg of both the geojson and bounding box
        svg.selectAll("path").data(data.features).enter().append("path")
            .attr("d", path)
            .style("fill", "#009926")
            .style("stroke-width", "1")
            .style("stroke", "#009926")

    });

</script>

Site URL: {{site.url}}
Data Source: {{site.url}}/articlesData/Abastecimento1.geojson



# References:
http://geojson.io/
https://cran.r-project.org/web/packages/geojsonio/README.html
http://www.dummies.com/how-to/content/how-to-create-a-data-frame-from-scratch-in-r.html
