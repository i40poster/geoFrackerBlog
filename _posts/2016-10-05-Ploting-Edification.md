---
layout: page
subheadline: "Exploring Plotting of GEOJSON"
title: "Ploting some polygons in GeoJSON"
teaser: "...DRAFT..."
categories:
  - R
tags:
  - Maps
  - SAO PAULO


comments: true
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetupV2)

# Let's explore data



```R

Edificacao = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_SE&arqTipo=Shapefile")
Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])
plot(Edificacao1)
```

## Exporting

```R

#force projection data(because we know that this is the used on the original data)
Edificacao1@proj4string@projargs <- paste0("+proj=utm"," +south  +zone=",23," +datum=","WGS84")
#transform
Edificacao1Projected <- spTransform(Edificacao1, CRS("+proj=longlat"))
name <- "Edification1"
writeOGR(Edificacao1Projected, paste0(name, '.geojson'), name, driver='GeoJSON')

```

# Extra commands
```R
geofracker.utm2decimalSouth <- function(data,zone,datum){
    #coordinates(newData) <- c("easting","northing")
    crs <- paste0("+proj=utm+zone=",zone,"+datum=",datum)
    data@proj4string@projargs <- paste0("+proj=utm"," +south  +zone=",zone," +datum=",datum)
    spTransform(data, CRS("+proj=longlat"))}


# Extra code:
# Lat, Long, Data
Edificacao1iDF <- data.frame(coordinates(Edificacao1inDegrees)[,2], coordinates(Edificacao1inDegrees)[,1], Edificacao1inDegrees$eq_id )

#Abastecimento1iDF <- data.frame(Abastecimento1iDF_Lat, Abastecimento1iDF_Long, Abastecimento1iDF$variable )
# Renaming
# http://stackoverflow.com/questions/7531868/how-to-rename-a-single-column-in-a-data-frame-in-r
colnames(Edificacao1iDF) = c("lat","long","data")
geojson_write(Edificacao1iDF, lat = 'lat', lon = 'long',file = "/home/rstudio/Edification1")

```

File Saved at: /home/rstudio/Abastecimento1
Copying it to your machine:

```bash
docker cp r_workbench:/home/rstudio/Edification1.geojson ~/Downloads/
```

You can test the generated file at: http://geojson.io/
> not working!



# D3.JS Rendering Section
> Not working!

<script src="https://d3js.org/d3.v3.min.js"></script>

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

    var width = 900,
        height = 900;

    console.log("{{site.url}}/articlesData/Edification1.geojson");

    var svg = d3.select("#viz").append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("class", "svg")
          .call(d3.behavior.zoom().on("zoom", function () {
          svg.attr("transform", "translate(" + d3.event.translate + ")" + " scale(" + d3.event.scale + ")")
        }))
        .append("g");
    console.log("{{site.url}}/articlesData/Edification1.geojson");



    d3.json("{{site.url}}/articlesData/Edification1.geojson", function(map) {
          var projection = d3.geo.mercator().scale(1).translate([0,0]).precision(0);
          var path = d3.geo.path().projection(projection);
          var bounds = path.bounds(map);

          var scale = .95 / Math.max((bounds[1][0] - bounds[0][0]) / width,
              (bounds[1][1] - bounds[0][1]) / height);
          var transl = [(width - scale * (bounds[1][0] + bounds[0][0])) / 2,
              (height - scale * (bounds[1][1] + bounds[0][1])) / 2];
          projection.scale(scale).translate(transl);

          svg.selectAll("path").data(map.features).enter().append("path")
            .attr("d", path)
            .style("fill", "none")
            .style("stroke", "black");
        });

        d3.json("https://i40poster.github.io/geoFrackerBlog/articlesData/Edification1.geojson", function(map) {
              var projection = d3.geo.mercator().scale(1).translate([0,0]).precision(0);
              var path = d3.geo.path().projection(projection);
              var bounds = path.bounds(map);

              var scale = .95 / Math.max((bounds[1][0] - bounds[0][0]) / width,
                  (bounds[1][1] - bounds[0][1]) / height);
              var transl = [(width - scale * (bounds[1][0] + bounds[0][0])) / 2,
                  (height - scale * (bounds[1][1] + bounds[0][1])) / 2];
              projection.scale(scale).translate(transl);

              vis.selectAll("path").data(map.features).enter().append("path")
                .attr("d", path)
                .style("fill", "none")
                .style("stroke", "black");
            });


</script>


# References:
<http://stackoverflow.com/questions/23953366/d3-large-geojson-file-does-not-show-draw-map-properly-using-projections>

geofracker.removeServiceBuildings

#geofracker.utm2decimalSouth

#geofracker.utm2decimalNorth
