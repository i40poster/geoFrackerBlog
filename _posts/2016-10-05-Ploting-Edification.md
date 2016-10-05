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

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetup)

# Let's explore data



```R

Edificacao = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=06_Habita%E7%E3o%20e%20Edifica%E7%E3o%5C%5CEdifica%E7%E3o%5C%5CShapefile%5C%5CSHP_edificacao_SE&arqTipo=Shapefile")
Edificacao1 <- readOGR(dsn=Edificacao$dir[1], layer=Edificacao$shapeclass[1])
plot(Edificacao1)

#Converting the Coordinates from UTM to Degrees
Edificacao1inDegrees <-  geofracker.utm2decimalSouth(Abastecimento1,23,"WGS84")
head(coordinates(Edificacao1inDegrees))
```

Expected Result:

```
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
name <- "Edification1"
writeOGR(Edificacao1inDegrees, paste0(name, '.geojson'), name, driver='GeoJSON')

#force projection data(because we know that this is the used on the original data)
Edificacao1@proj4string@projargs <- paste0("+proj=utm"," +south  +zone=",23," +datum=","WGS84")
#transform
spTransform(Edificacao1, CRS("+proj=longlat"))
writeOGR(Edificacao1, paste0(name, '.geojson'), name, driver='GeoJSON')

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
20160821.SampleGeoJSON.png
![GEOJSON Sample]({{site.url}}/images/20160821.SampleGeoJSON.png)



# D3.JS Rendering Section

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

/*
    $.get('https://raw.githubusercontent.com/i40poster/geoFrackerBlog/master/articlesData/Abastecimento1.geojson',
                      function(data) {
                        console.log(data);
                                   }
                     )
                     */

    var svg = d3.select("#viz").append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("class", "svg");

/*
    // load geojson and do stuff in a callback function...
    //Fixed projection to be closer to what we see on GeoSampa*/
    console.log("{{site.url}}/articlesData/Edification1.geojson");
/*
    //https://raw.githubusercontent.com/alignedleft/d3-book/master/chapter_12/*/

/*
    //this not works on github pages.. not sure why yet
    //d3.json("{{site.url}}/articlesData/Abastecimento1.geojson",
    d3.json("https://raw.githubusercontent.com/i40poster/geoFrackerBlog/master/articlesData/Abastecimento1.geojson",*/

    d3.json("{{site.url}}/articlesData/Edification1.geojson",
    function(error, data){
        /*// console.log the data
        alert(error);*/
        console.log(data);

        /*// create a unit projection*/
        var projection = d3.geo.mercator()
            .scale(1)
            .translate([0,0]);

        /*// create a path generator.*/
        console.log( d3.geo.path());
        var path = d3.geo.path()
            .projection(projection)
            .pointRadius(function(d) {
              return 2;
          /*  //  return d.properties.mag;*/
            });

        /*// compute bounds of a point of interest, then derive scale and translate*/
        var b = path.bounds(data),
            s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height),
            t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2];

      /*  // update the projection to use computed scale and translate....*/
        projection
            .scale(s)
            .translate(t);



        svg.append("rect")
            .attr('width', width)
            .attr('height', height)
            .style('stroke', 'black')
            .style('fill', '#dfd');


        svg.selectAll("path").data(data.features).enter().append("path")
            .attr("d", path)
            .style("fill", "#009926")
            .style("stroke-width", "1")
            .style("stroke", "#009926")

    });

    /* code reused from the following stackoverflow question:
                  http://stackoverflow.com/questions/14492284/center-a-map-in-d3-given-a-geojson-object
 // draw the svg of both the geojson and bounding box
// calculate and draw a bounding box for the geojson
                  */

</script>

# References:

geofracker.removeServiceBuildings

#geofracker.utm2decimalSouth

#geofracker.utm2decimalNorth
