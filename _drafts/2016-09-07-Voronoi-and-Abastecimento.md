---
layout: page
#
# Content
#
subheadline: "Exploring the location of São Paulo services"
title: "Let's play with voronoi and São Paulo data(DRAFT)"
teaser: "Voronoi, and some São Paulo data."
categories:
  - R
tags:
  - Maps
  - SAO PAULO
  - D3.js
  - GeoJSON
  - Polygons
  - Voronoi
#
# Styling
#
image:
  #header: ""
  #thumb: "you-can-delete-me-thumb.png"
  #homepage: "you-can-delete-me-homepage.png"
  #caption: "Caption?"
  #url: "http://phlow.de/"

comments: true
---

Before we start, did you have followed the setup steps described at [here]({{site.url}}/RStudioSetup)

# Let's explore data

## Source of our data for this sample:

Source: http://geosampa.prefeitura.sp.gov.br/


- [ ] Loading some data: Points(Abastecimento)
- [ ] Loading Neiboorhoods Maps:

```R
Population = downloadAndUnzipShp("http://geosampa.prefeitura.sp.gov.br/PaginasPublicas/downloadArquivoOL.aspx?orig=DownloadCamadas&arq=02_Popula%E7%E3o%5C%5CDensidade%20Demogr%E1fica%5C%5CShapefile%5C%5CSAD69_SHP_densidade_demografica_2010&arqTipo=Shapefile")
Population1 <- readOGR(dsn=Population$dir[1], layer=Population$shapeclass[1])



#Converting the Coordinates from UTM to Degrees
Population1inDegrees <- utm2decimalSouth(Population1,23,"WGS84")


geojson_write( Population1inDegrees, lat = Population1inDegrees[,2], lon = Population1inDegrees[,1],file = "/home/rstudio/populationSP")

```



File Saved at:

/home/rstudio/populationSP.geojson

Copying it to your machine:

```bash
docker cp r_workbench:/home/rstudio/populationSP.geojson ~/Downloads/
```

You can test the generated file at: http://geojson.io/


# D3.JS Rendering Section

**Attention**: This page loads 50MB of data to render the area below, be adviced on data consumption when accessing it.

<meta charset="utf-8">
<style>

.land {
  fill: #ddd;
}

.state-borders {
  fill: none;
  stroke: #fff;
}

.airport-arc {
  fill: none;
}

.airport:hover .airport-arc {
  stroke: #f00;
}

.airport-cell {
  fill: none;
  stroke: #000;
  stroke-opacity: 0.1;
  pointer-events: all;
}

</style>
<svg width="960" height="600"></svg>
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://d3js.org/topojson.v1.min.js"></script>
<script>

var svg = d3.select("svg"),
    width = +svg.attr("width"),
    height = +svg.attr("height");

var projection = d3.geoAlbers()
    .translate([width / 2, height / 2])
    .scale(1280);

var radius = d3.scaleSqrt()
    .domain([0, 100])
    .range([0, 14]);

var path = d3.geoPath()
    .projection(projection)
    .pointRadius(2.5);

var voronoi = d3.voronoi()
    .extent([[-1, -1], [width + 1, height + 1]]);

d3.queue()
    .defer(d3.json, "http://bl.ocks.org/mbostock/raw/4090846/us.json")
    .defer(d3.csv, "http://bl.ocks.org/mbostock/raw/7608400/e5974d9bba45bc9ab272d98dd7427567aafd55bc/airports.csv", typeAirport)
    .defer(d3.csv, "http://bl.ocks.org/mbostock/raw/7608400/e5974d9bba45bc9ab272d98dd7427567aafd55bc/flights.csv", typeFlight)
    .await(ready);

function ready(error, us, airports, flights) {
  if (error) throw error;

  var airportByIata = d3.map(airports, function(d) { return d.iata; });

  flights.forEach(function(flight) {
    var source = airportByIata.get(flight.origin),
        target = airportByIata.get(flight.destination);
    source.arcs.coordinates.push([source, target]);
    target.arcs.coordinates.push([target, source]);
  });

  airports = airports
      .filter(function(d) { return d.arcs.coordinates.length; });

  svg.append("path")
      .datum(topojson.feature(us, us.objects.land))
      .attr("class", "land")
      .attr("d", path);

  svg.append("path")
      .datum(topojson.mesh(us, us.objects.states, function(a, b) { return a !== b; }))
      .attr("class", "state-borders")
      .attr("d", path);

  svg.append("path")
      .datum({type: "MultiPoint", coordinates: airports})
      .attr("class", "airport-dots")
      .attr("d", path);

  var airport = svg.selectAll(".airport")
    .data(airports)
    .enter().append("g")
      .attr("class", "airport");

  airport.append("title")
      .text(function(d) { return d.iata + "\n" + d.arcs.coordinates.length + " flights"; });

  airport.append("path")
      .attr("class", "airport-arc")
      .attr("d", function(d) { return path(d.arcs); });

  airport.append("path")
      .data(voronoi.polygons(airports.map(projection)))
      .attr("class", "airport-cell")
      .attr("d", function(d) { return d ? "M" + d.join("L") + "Z" : null; });
}

function typeAirport(d) {
  d[0] = +d.longitude;
  d[1] = +d.latitude;
  d.arcs = {type: "MultiLineString", coordinates: []};
  return d;
}

function typeFlight(d) {
  d.count = +d.count;
  return d;
}

</script>

# References:
http://geojson.io/
https://cran.r-project.org/web/packages/geojsonio/README.html
http://www.dummies.com/how-to/content/how-to-create-a-data-frame-from-scratch-in-r.html
http://bl.ocks.org/sgruhier/1d692762f8328a2c9957
