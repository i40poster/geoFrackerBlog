---
layout: page
#
# Content
#
subheadline: "Exploring Population density data and GeoJSON polygons"
title: "Working City Population Data"
teaser: "A brief exploration on R with GeoJSON and D3.js to render colored polygons"
categories:
  - R
tags:
  - Maps
  - SAO PAULO
  - D3.js
  - GeoJSON
  - Polygons
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

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetup)

# Let's explore data

## Source of our data for this sample:
Source: http://geosampa.prefeitura.sp.gov.br/


```R
Population = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=02_Popula%E7%E3o%5C%5CDensidade%20Demogr%E1fica%5C%5CShapefile%5C%5CSAD69_SHP_densidade_demografica_2010&arqTipo=Shapefile")
Population1 <- readOGR(dsn=Population$dir[1], layer=Population$shapeclass[1])



#Converting the Coordinates from UTM to Degrees
Population1inDegrees <- utm2decimalSouth(Population1,23,"WGS84")


geojson_write( Population1inDegrees, lat = Population1inDegrees[,2], lon = Population1inDegrees[,1],file = "/home/rstudio/populationSP")

```


File Saved at: /home/rstudio/populationSP.geojson
Copying it to your machine:

```bash
docker cp r_workbench:/home/rstudio/populationSP.geojson ~/Downloads/
```
You can test the generated file at: http://geojson.io/


# D3.JS Rendering Section

**Attention**: This page loads 50MB of data to render the area below, be adviced on data consumption when accessing it.

<script src="https://d3js.org/d3.v3.min.js"></script>
<style>
#viz {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
}
</style>

<div id="viz"></div>
<script>
    var width = 630,
        height = 900;
    console.log("{{site.url}}/articlesData/populationSP.geojson");

    var svg = d3.select("#viz").append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("class", "svg");

    d3.json("{{site.url}}/articlesData/populationSP.geojson",
    function(error, data){
        var projection = d3.geo.mercator()
            .scale(1)
            .translate([0,0]);

        var path = d3.geo.path()
            .projection(projection)
            .pointRadius(function(d) {
              return 2;
            });


        var b = path.bounds(data),
            s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height),
            t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2];

        projection
            .scale(s)
            .translate(t);

        svg.append("rect")
            .attr('width', width)
            .attr('height', height)
            .style('stroke', 'black')
            .style('fill', '#efe');


        svg.selectAll("path").data(data.features).enter().append("path")
            .attr("d", path)
            .style("fill", function(d) {
              /* There is null data - Not Available data
              Ranges:
              [0,55] - color 1 #fee5d9
              (55,126] - color 2 #fcae91
              (126, 204] - color 3  #fb6a4a
              (204,+infinitum] - color 4 #cb181d
              NA - Color 5  #5e3c99
              */
              if(  d.properties.habit_hect == null)
                return '#5e3c99';
              else if(  d.properties.habit_hect >= 0 && d.properties.habit_hect <= 55)
                return  '#fee5d9';
              else if(  d.properties.habit_hect > 55 && d.properties.habit_hect <= 126)
                  return  '#fcae91';
              else if(  d.properties.habit_hect > 126 && d.properties.habit_hect <= 204)
                  return  '#fb6a4a';
              else if(  d.properties.habit_hect > 204 )
                  return  '#cb181d';
              else
                return '#5e3c99';

            } )
            .style("stroke-width", "1")
            .style("stroke", "none")

    });

</script>

# References:
http://geojson.io/
https://cran.r-project.org/web/packages/geojsonio/README.html
http://www.dummies.com/how-to/content/how-to-create-a-data-frame-from-scratch-in-r.html
