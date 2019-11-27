<html>
<head>
 <meta charset="utf-8">
 <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no">
 <title>ArcGIS JavaScript Tutorials: Find places</title>
 <style>
   html, body, #viewDiv {
     padding: 0;
     margin: 0;
     height: 100%;
     width: 100%;
   }
 </style>
 
 <link rel="stylesheet" href="https://js.arcgis.com/4.13/esri/css/main.css">
 <script src="https://js.arcgis.com/4.13/"></script>
 
 <script>  
   require([
     "esri/Map",
     "esri/views/MapView",
     "esri/tasks/Locator",
     "esri/Graphic"
   ], function(Map, MapView, Locator, Graphic) {

     var map = new Map({
       basemap: "topo-vector"
     });
     
     var view = new MapView({
       container: "viewDiv",
       map: map,
       center: [-78.644257,35.787743],
       zoom: 13
     });
     
     var places = ["Coffee shop", "Gas station", "Food", "Hotel", "Parks and Outdoors"];
     
     var select = document.createElement("select","");
      select.setAttribute("class", "esri-widget esri-select");
      select.setAttribute("style", "width: 175px; font-family: Avenir Next W00; font-size: 1em");
     places.forEach(function(p){
       var option = document.createElement("option");
       option.value = p;
       option.innerHTML = p;
       select.appendChild(option);
     });
     
     view.ui.add(select, "top-right");
     
     var locator = new Locator({
        url: "http://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer"
     });
 
     // Find places and add them to the map
     function findPlaces(category, pt) {
       locator.addressToLocations({
         location: pt,
         categories: [category],
         maxLocations: 25,
         outFields: ["Place_addr", "PlaceName"]
       })
       .then(function(results) {
         view.popup.close();
         view.graphics.removeAll();
         results.forEach(function(result){
           view.graphics.add(
             new Graphic({
               attributes: result.attributes,
               geometry: result.location,
               symbol: {
                type: "simple-marker",
                color: "#000000",
                size: "12px",
                outline: {
                  color: "#ffffff",
                  width: "2px"
                }
               },
               popupTemplate: {
                 title: "{PlaceName}",
                 content: "{Place_addr}"
               }
            }));
         });
       });
     }
     
     // Search for places in center of map
     findPlaces(select.value, view.center);

     // Listen for category changes and find places
     select.addEventListener('change', function (event) {
       findPlaces(event.target.value, view.center);
     });