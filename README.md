# nostra-trip
the example for use Nostra Javascript API.

> npm start
> feel free dev Nostra-trip on: http://127.0.0.1/src/index.html 

1. สร้าง event document ready และนำ code ในขั้นตอนถัดๆ ไปใส่ function นี้
=======================================================================================

```javascript
$(document).ready(function () {

});
```

2. ประกาศตัวตัวแปร 
=======================================================================================

```javascript
var basemap, speialLayer,
    locations = [], latCurrent,
    lonCurrent, pointLayer, routeLayer,
    gpMarkerPoint = null,
    enableMapClick = false,
    routeOption;
```
   
   


3. เรียกใช้งาน class nostra.maps.Map
=======================================================================================
```javascript
nostra.onready = function () {
nostra.config.Language.setLanguage(nostra.language.L);

    map = new nostra.maps.Map("divMap", {
        id: "mapTest",
        logo: true,
        scalebar: true,
        basemap: "streetmap",
        slider: true,
        level: 10,
        lat: 13.7262435,
        lon: 100.5373896
    });

    initGraphicLayer();
    initYourLocation();
};
```

  

4. สร้าง initGraphicLayer() เพื่อสร้าง graphic layer
=======================================================================================

```javascript
function initGraphicLayer() {
    routeLayer = new nostra.maps.layers.GraphicsLayer(map, {
        id: "routeLayer"
    });
    map.addLayer(routeLayer);

    pointLayer = new nostra.maps.layers.GraphicsLayer(map, {
        id: "pointLayer"
    });
    map.addLayer(pointLayer);

}
```



5. สร้าง initGraphicLayer() เพื่อสร้าง pin แสดงตำแหน่ง current location จากเครื่อง  
=======================================================================================

```javascript
function initYourLocation() {
    if (navigator.geolocation) {
    	navigator.geolocation.getCurrentPosition(function (position) {

    	var marker = new nostra.maps.symbols.Marker({
            url: "/src/images/pin_margin@2x.png",
            width: 60,
            height: 60,
            draggable: false,
            label: "Start Location"
        });
            
        var locaitonMarker = pointLayer.addMarker(position.coords.latitude, position.coords.longitude, marker);

        var obj = {
            id: locations.length + 1,
            name: "Your location",
            description: "",
            lat: position.coords.latitude,
            lon: position.coords.longitude,
            gpMaker: locaitonMarker
        }
            locations.push(obj);
            createLocationList();
    	});
    } else {
        alert("Geolocation is not supported by this browser.");
    }

}
```




6. สร้าง createLocationList() เพื่อสร้างรายการ location ที่ panel list
=======================================================================================

```javascript
 function createLocationList() {
    var mainDiv = document.getElementById("locationList");
    mainDiv.innerHTML = "";

    locations.map(function (location, index) {
        var divMain = document.createElement("div");
        divMain.className = "row list-location"

        var divNumber = document.createElement("div");
        divNumber.className = "col-md-2";
        divNumber.innerHTML = index + 1;
        divMain.appendChild(divNumber);

        var divName = document.createElement("div");
        divName.className = "col-md-8";
        divName.innerHTML = location.name;
        divMain.appendChild(divName);

        var divRemove = document.createElement("div");
        divRemove.className = "col-md-2";
        divMain.appendChild(divRemove);

        if (index != 0) {
            var btnRemove = document.createElement("button");
            btnRemove.className = "btn btn-danger btn-sm";
            btnRemove.innerHTML = "X";
            divRemove.appendChild(btnRemove);

            btnRemove.onclick = function () {
                removeLocation(index)
            }
        }
        mainDiv.appendChild(divMain);
    });
}
```




9. สร้าง removeLocation() เพื่อลบตำแหน่ง และลบ pin ของ location  
=======================================================================================

```javascript
function removeLocation(index) {
    pointLayer.removeGraphic(locations[index].gpMaker);
    locations = locations.filter(function (obj, findex) {
        return index != findex
    })
    routeLayer.clear();
    createLocationList();
}
```

  


7. กำหนด special layer ต่างๆ เมื่อเกิดการคลิกปุ่มใน layer panel 
=======================================================================================

```javascript
document.getElementById("btnStreetMap").onclick = function () {
    basemap = new nostra.maps.layers.StreetMap(map);
    map.addLayer(basemap);
}

document.getElementById("btnImagery").onclick = function () {
    basemap = new nostra.maps.layers.Imagery(map);
    map.addLayer(basemap);
}
document.getElementById("btnOpenStreetMap").onclick = function () {
    basemap = new nostra.maps.layers.OpenStreetMap(map);
    map.addLayer(basemap);
}

document.getElementById("btnNOSTRAGuide").onclick = function () {
    speialLayer = new nostra.maps.layers.NOSTRAGuide(map);
    map.addLayer(speialLayer);
}

document.getElementById("btnNOSTRATraffic").onclick = function () {
    speialLayer = new nostra.maps.layers.NOSTRATraffic(map);
    map.addLayer(speialLayer);
}

document.getElementById("btnWeatherForecast").onclick = function () {
    speialLayer = new nostra.maps.layers.WeatherForecast(map);
    map.addLayer(speialLayer);
}
```




8. กำหนดการทำงานของปุ่ม create location 
=======================================================================================

```javascript
document.getElementById("btnCreateLocation").onclick = function () {
    document.getElementById("createPinPanal").style.display = "block";
    document.getElementById("btnCreateLocation").style.display = "none";
    document.getElementById("locationNameInput").value = "";
    document.getElementById("descriptionInput").value = "";
    enableMapClick = true;
}
```



8. เพิ่ม event เพื่อบันทึกตำแหน่ง location เมื่อมีการคลิกที่ map
=======================================================================================

```javascript
- nostra.onready = function () {
- nostra.config.Language.setLanguage(nostra.language.L);
- 
- map = new nostra.maps.Map("divMap", {
- id: "mapTest",
- logo: true,
- scalebar: true,
- basemap: "streetmap",
- slider: true,
- level: 10,
- lat: 13.7262435,
- lon: 100.5373896
- });
- 
- initGraphicLayer();
- initYourLocation();
- 

+ map.events.click = function (evt) {
+ 	handleAddLocation(evt.mapPoint.getLatitude(), evt.mapPoint.getLongitude());
+ }

- };
```

  
9. สร้าง addLocation() เพื่อบันทึกตำแหน่ง และสร้าง pin ของ location  
=======================================================================================

```javascript
function addLocation(lat, lon) {
if (gpMarkerPoint == null && enableMapClick) {
    latCurrent = lat;
    lonCurrent = lon;

callIdentifyService();

var pointMarker = new nostra.maps.symbols.Marker({
    width: 60,
    height: 60,
    draggable: true
});

pointMarker.onDragEnd = function (point) {
    latCurrent = point.lat;
    lonCurrent = point.lon;
    callIdentifyService(point.lat, point.lon);
}

var gp = pointLayer.addMarker(latCurrent, lonCurrent, pointMarker);
gpMarkerPoint = gp;
}
}
```

  

10. สร้าง function callIdentifyService() เพื่อนำข้อมุลที่ได้มาใช้เป็นชื่อของตำแหน่ง
=======================================================================================

```javascript
function callIdentifyService() {
    locationTextLoading(true);
    var key = "{API-KEY}";
    var url = "https://api.nostramap.com/Service/V2/Location/Identify?key=" + key + "&lat=" + latCurrent + "&lon=" + lonCurrent
    
    $.ajax({
        url: url,
        dataType: 'jsonp',
        success: function (res) {
            locationTextLoading(false);
            if (res != null) {
                document.getElementById("locationNameInput").value = res.results[0].Name_L;
            }
        },
        error: function (err) {
            locationTextLoading(false);
            console.log("nostra identify error: ", err);
        }
    });
}
```

    


10. กำหนดการทำงาน textbox locationNameInput
=======================================================================================

```javascript
function locationTextLoading(isLoad) {
    document.getElementById("locationNameInput").value = "";
    document.getElementById("locationNameInput").placeholder = (isLoad) ? "name of location loading..." : "";
}
```

11. กำหนดการทำงานเมื่อสร้าง location ใหม่
=======================================================================================

```javascript
document.getElementById("btnAddLocation").onclick = function () {
document.getElementById("createPinPanal").style.display = "none";
document.getElementById("btnCreateLocation").style.display = "block";

var name = document.getElementById("locationNameInput").value;
var desc = document.getElementById("descriptionInput").value;

enableMapClick = false;
pointLayer.removeGraphic(gpMarkerPoint);
gpMarkerPoint = null;

var callout = new nostra.maps.CustomCallout({
    content: name,
    width: 100,
    height: 100,
});

var marker = new nostra.maps.symbols.Marker({
    url: "https://png.pngtree.com/png-clipart/20190516/original/pngtree-vector-location-icon-png-image_3778133.jpg",
    width: 60,
    height: 60,
    draggable: false,
    customCallout: callout
});
var locaitonMarker = pointLayer.addMarker(latCurrent, lonCurrent, marker);

var obj = {
    id: locations.length + 1,
    name: name,
    description: desc,
    lat: latCurrent,
    lon: lonCurrent,
    gpMaker: locaitonMarker
}
locations.push(obj);
createLocationList();
}
```

    

12. กำหนดค่าเริ่มต้นของ route
=======================================================================================
 ```javascript
  - nostra.onready = function () {
  - nostra.config.Language.setLanguage(nostra.language.L);
  - 
  
  + routeOption = {
  + impedance: nostra.services.network.impedance.TIME,
  + mode: nostra.services.network.mode.CAR,
  + tollroad: false,
  + findBestSequence: false,
  + preserveLastStop: false,
  + tollroad: false
  + }
  -
  
  - map = new nostra.maps.Map("divMap", {
  - id: "mapTest",
  - logo: true,
  - scalebar: true,
  - basemap: "streetmap",
  - slider: true,
  - level: 10,
  - lat: 13.7262435,
  - lon: 100.5373896
  - });
  - 
  - initGraphicLayer();
  - initYourLocation();
  - 
  - map.events.click = function (evt) {
  -     handleAddLocation(evt.mapPoint.getLatitude(), evt.mapPoint.getLongitude());
  - }
  - };
  ```

  
13. กำหนดการทำงานเมื่อทำการกด route เส้นทาง
=======================================================================================

```javascript
document.getElementById("btnRoute").onclick = function () {
    direction();
}
```


14. สร้าง เพื่อวาดเส้นทาง และคำนวณเส้นทาง
=======================================================================================

```javascript
function direction() {
    routeLayer.clear();

    var route = new nostra.services.network.route();
    route.clearStopPoint();
    route.country = "TH";
    route.returnedRouteDetail = true;
    route.preserveFirstStop = true;

    route.impedance = routeOption.impedance;
    route.mode = routeOption.mode;
    route.tollroad = routeOption.tollroad;
    route.findBestSequence = routeOption.findBestSequence;
    route.preserveLastStop = routeOption.preserveLastStop;

    locations.map(function (location, index) {
    var stop = new nostra.services.network.stopPoint({
        name: location.name,
        lat: location.lat,
        lon: location.lon
    });
    route.addStopPoint(stop);
});

route.solve(function (solveResult) {
    if (solveResult.errorMessage == null) {
        routeLayer.addRoute(solveResult, '#007aff', 1);
        reOrderLocaitons(solveResult.nostraResults.stops);
        showRouteResult(solveResult.nostraResults.totalLength, solveResult.nostraResults.totalTime);
    }
});

}
```



15. เรียงลำดับของตำแหน่ง location
=======================================================================================

```javascript
function reOrderLocaitons(stops) {
    var newLocations = [];
    stops.map(function (stop) {
        var result = locations.filter(function (location) {
            return location.id == stop.sequence
        });
        newLocations.push(result[0]);
    });
    locations = newLocations;
    createLocationList();
}
```

    

16.  นำข้อมูลที่ได้จากการ route มาคำนวณระยะทางเวลา
=======================================================================================

```javascript
function showRouteResult(distance, time) {
    var calTime = parseFloat((time / 60)).toFixed(0) + " hr. " + parseFloat((time % 60)).toFixed(0) + " min.";
    var calDistance = parseFloat(distance / 1000).toFixed(2) + ' Km';

    document.getElementById("divTime").innerText = calTime;
    document.getElementById("divDistance").innerText = calDistance;

    if (routeOption.mode == nostra.services.network.mode.CAR || routeOption.mode == nostra.services.network.mode.MOTORCYCLE) {
            var calFuelConsumption = (parseFloat((distance / 1000) / 14)).toFixed(2) + " L.";
            var calEstFuelCost = "฿" + (parseFloat(((distance / 1000) / 14) * 35.36).toFixed(2));
    
    
        document.getElementById("divFuelConsumption").innerText = calFuelConsumption;
        document.getElementById("divEstFuelCost").innerText = calEstFuelCost;
    } else {
        document.getElementById("divFuelConsumption").innerText = "";
        document.getElementById("divEstFuelCost").innerText = "";
    }
}
```

    

17. เพิ่ม option route
=======================================================================================

```javascript
document.getElementById("btnCar").onclick = function () {
    routeOption.mode = nostra.services.network.mode.CAR;
    direction();
}

document.getElementById("btnBicycle").onclick = function () {
    routeOption.mode = nostra.services.network.mode.BICYCLE;
    direction();
}

document.getElementById("btnWalk").onclick = function () {
    routeOption.mode = nostra.services.network.mode.WALK;
    direction();
}

document.getElementById("btnMotercycle").onclick = function () {
    routeOption.mode = nostra.services.network.mode.MOTORCYCLE;
    direction();
}

document.getElementById("switchFindBestSeq").onchange = function (evt) {
    routeOption.findBestSequence = evt.target.checked;
    direction();
}

document.getElementById("switchPreserveLastStop").onchange = function (evt) {
    routeOption.preserveLastStop = evt.target.checked;
    direction();
}

document.getElementById("switchTollRoad").onchange = function (evt) {
    routeOption.tollroad = evt.target.checked;
    direction();
}
```


