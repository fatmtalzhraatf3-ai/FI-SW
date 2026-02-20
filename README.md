<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>مشروع مساحي متقدم</title>

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<!-- Turf.js لحساب المساحات -->
<script src="https://cdn.jsdelivr.net/npm/@turf/turf@6/turf.min.js"></script>

<style>
body { margin:0; font-family:Arial; background:#111; color:#fff;}
header{padding:15px; text-align:center; background:#222;}
#controls{padding:10px; background:#222; display:flex; flex-wrap:wrap; justify-content:center; gap:5px;}
input, button{padding:6px; border-radius:4px; border:none;}
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

<header><h1>مشروع مساحي متقدم</h1></header>

<div id="controls">
<input type="text" id="lat" placeholder="خط العرض">
<input type="text" id="lng" placeholder="خط الطول">
<input type="number" id="depth" placeholder="عمق الحفر/الردم (م)" value="0.5" step="0.1">
<button onclick="addMarker()">أضف نقطة</button>
<button onclick="drawTraverse()">رسم الترافيرس</button>
<button onclick="drawPolygon()">رسم المنطقة</button>
<button onclick="calculateVolume()">احسب الحفر/الردم</button>
</div>

<div id="map"></div>
<div id="table-container"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
// إنشاء الخريطة
var map = L.map('map').setView([30.0444,31.2357],15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
attribution:'&copy; OpenStreetMap contributors'
}).addTo(map);

// المتغيرات
var markers = [];
var traverseLine = null;
var polygonLayer = null;

// إضافة نقطة
function addMarker(){
    var lat=parseFloat(document.getElementById('lat').value);
    var lng=parseFloat(document.getElementById('lng').value);
    if(isNaN(lat)||isNaN(lng)){ alert("ادخلي إحداثيات صحيحة"); return; }
    L.marker([lat,lng]).addTo(map).bindPopup("نقطة جديدة").openPopup();
    markers.push([lat,lng]);
    map.setView([lat,lng],16);
    updateTable();
}

// رسم الترافيرس
function drawTraverse(){
    if(markers.length<2){ alert("يجب إدخال نقطتين على الأقل"); return; }
    if(traverseLine) map.removeLayer(traverseLine);
    traverseLine=L.polyline(markers,{color:'red'}).addTo(map);
    let totalDistance=0;
    for(let i=0;i<markers.length-1;i++){
        totalDistance += map.distance(markers[i],markers[i+1]);
    }
    alert("طول الترافيرس: "+totalDistance.toFixed(2)+" متر");
}

// رسم Polygon (المنطقة)
function drawPolygon(){
    if(markers.length<3){ alert("أدخل ثلاث نقاط على الأقل"); return; }
    if(polygonLayer) map.removeLayer(polygonLayer);
    polygonLayer=L.polygon(markers,{color:'green', fillOpacity:0.4}).addTo(map).bindPopup("المنطقة");
    map.fitBounds(polygonLayer.getBounds());
}

// حساب الحفر/الردم
function calculateVolume(){
    if(!polygonLayer){ alert("ارسم المنطقة أولا"); return; }
    var depth=parseFloat(document.getElementById('depth').value);
    var coords=polygonLayer.getLatLngs()[0].map(p=>[p.lng,p.lat]);
    coords.push(coords[0]);
    var poly=turf.polygon([coords]);
    var area=turf.area(poly);
    var volume=area*depth;
    alert("المساحة: "+area.toFixed(2)+" م²\nالحفر/الردم: "+volume.toFixed(2)+" م³");
    updateTable();
}

// تحديث الجدول
function updateTable(){
    var tableHTML=`<table><tr><th>نقطة</th><th>خط العرض</th><th>خط الطول</th></tr>`;
    markers.forEach((p,i)=>{
        tableHTML+=`<tr><td>${i+1}</td><td>${p[0]}</td><td>${p[1]}</td></tr>`;
    });
    tableHTML+=`</table>`;
    document.getElementById('table-container').innerHTML=tableHTML;
}
</script>

</body>
</html>
