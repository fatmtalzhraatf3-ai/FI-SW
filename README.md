<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>مشروع GIS متقدم</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@turf/turf@6/turf.min.js"></script>
<style>
body { margin:0; font-family:Arial; background:#111; color:#fff;}
header{padding:15px; text-align:center; background:#222;}
#controls{padding:10px; background:#222; text-align:center;}
input, button{padding:6px; margin:3px; border-radius:4px; border:none;}
input{width:120px;}
button{background:#0a84ff; color:#fff; cursor:pointer;}
button:hover{background:#0066cc;}
#map{height:60vh; width:100%;}
table{margin:10px auto; border-collapse:collapse; width:90%; color:#fff;}
th,td{border:1px solid #555; padding:5px; text-align:center;}
th{background:#333;} td{background:#222;}
</style>
</head>
<body>

<header><h1>مشروع GIS متقدم</h1></header>

<div id="controls">
<input type="text" id="lat" placeholder="خط العرض">
<input type="text" id="lng" placeholder="خط الطول">
<input type="number" id="depth" placeholder="عمق الحفر (م)" value="0.5" step="0.1">
<button onclick="addMarker()">أضف نقطة</button>
<button onclick="drawTraverse()">رسم الترافيرس</button>
<button onclick="drawPolygon()">رسم المنطقة</button>
<button onclick="calculateVolume()">احسب الحفر/الردم</button>
</div>

<div id="map"></div>
<div id="table-container"></div>

<script>
var map = L.map('map').setView([30.0444,31.2357],15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
attribution:'&copy; OpenStreetMap contributors'
}).addTo(map);

var markers=[], traverseLine, polygonLayer;

function addMarker(){
    var lat=parseFloat(document.getElementById('lat').value);
    var lng=parseFloat(document.getElementById('lng').value);
    if(isNaN(lat)||isNaN(lng)){alert("ادخلي إحداثيات صحيحة"); return;}
    L.marker([lat,lng]).addTo(map).bindPopup("نقطة جديدة").openPopup();
    markers.push([lat,lng]);
    map.setView([lat,lng],16);
    updateTable();
}

function drawTraverse(){
    if(markers.length<2){alert("يجب إدخال نقطتين على الأقل"); return;}
    if(traverseLine) map.removeLayer(traverseLine);
    traverseLine=L.polyline(markers,{color:'red'}).addTo(map);
    let total=0;
    for(let i=0;i<markers.length-1;i++){total+=map.distance(markers[i],markers[i+1]);}
    alert("طول الترافيرس: "+total.toFixed(2)+" متر");
}

function drawPolygon(){
    if(markers.length<3){alert("أدخل ثلاث نقاط على الأقل"); return;}
    if(polygonLayer) map.removeLayer(polygonLayer);
    polygonLayer=L.polygon(markers,{color:'green',fillOpacity:0.4}).addTo(map).bindPopup("المنطقة");
    map.fitBounds(polygonLayer.getBounds());
}

function calculateVolume(){
    if(!polygonLayer){alert("ارسم المنطقة أولا!"); return;}
    var depth=parseFloat(document.getElementById('depth').value);
    var coords=polygonLayer.getLatLngs()[0].map(p=>[p.lng,p.lat]);
    coords.push(coords[0]);
    var poly=turf.polygon([coords]);
    var area=turf.area(poly);
    var volume=area*depth;
    alert("المساحة: "+area.toFixed(2)+" م²\nالحفر/الردم: "+volume.toFixed(2)+" م³");
    updateTable();
}

function updateTable(){
    var table=`<table><tr><th>نقطة</th><th>خط العرض</th><th>خط الطول</th></tr>`;
    markers.forEach((p,i)=>{table+=`<tr><td>${i+1}</td><td>${p[0]}</td><td>${p[1]}</td></tr>`;});
    table+=`</table>`;
    document.getElementById('table-container').innerHTML=table;
}
</script>

</body>
</html>
