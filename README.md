<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>مشروع مساحي متقدم</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
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
table{margin:10px auto; border-collapse:collapse; width:95%; color:#fff;}
th,td{border:1px solid #555; padding:5px; text-align:center;}
th{background:#333;} td{background:#222;}
</style>
</head>
<body>

<header><h1>مشروع مساحي متقدم</h1></header>

<div id="controls">
<input type="text" id="lat" placeholder="خط العرض">
<input type="text" id="lng" placeholder="خط الطول">
<input type="number" id="depth" placeholder="عمق الحفر (م)" value="0.5" step="0.1">
<input type="number" id="fillHeight" placeholder="ارتفاع الردم (م)" value="0.3" step="0.1">
<button onclick="addMarker()">أضف نقطة</button>
<button onclick="drawTraverse()">رسم الترافيرس</button>
<button onclick="drawPolygon()">رسم المنطقة</button>
<button onclick="calculateVolume()">احسب الحفر/الردم + شبكة</button>
</div>

<div id="map"></div>
<div id="table-container"></div>

<script>
// إعداد الخريطة
var map = L.map('map').setView([30.0444,31.2357],15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
    attribution:'&copy; OpenStreetMap contributors'
}).addTo(map);

var markers=[], traverseLine=null, polygonLayer=null, gridLayer=null;

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
    let total=0;
    for(let i=0;i<markers.length-1;i++){
        total+=map.distance(markers[i],markers[i+1]);
    }
    alert("طول الترافيرس: "+total.toFixed(2)+" متر");
}

// رسم Polygon (المنطقة)
function drawPolygon(){
    if(markers.length<3){ alert("أدخل ثلاث نقاط على الأقل"); return; }
    if(polygonLayer) map.removeLayer(polygonLayer);
    polygonLayer=L.polygon(markers,{color:'green', fillOpacity:0.4}).addTo(map).bindPopup("المنطقة");
    map.fitBounds(polygonLayer.getBounds());
}

// حساب الحفر/الردم + إنشاء الشبكة
function calculateVolume(){
    if(!polygonLayer){ alert("ارسم المنطقة أولا"); return; }
    var depth=parseFloat(document.getElementById('depth').value);
    var fillHeight=parseFloat(document.getElementById('fillHeight').value);

    var bounds=polygonLayer.getBounds();
    var gridStepLat=(bounds.getNorth()-bounds.getSouth())/5; // تقسيم المنطقة 5x5
    var gridStepLng=(bounds.getEast()-bounds.getWest())/5;

    if(gridLayer) map.removeLayer(gridLayer);
    gridLayer=L.layerGroup();

    let tableHTML=`<table>
    <tr><th>خلية</th><th>إحداثيات مركز الخلية</th><th>مساحة م²</th><th>حجم الحفر م³</th><th>حجم الردم م³</th></tr>`;

    let cellCount=0;
    for(let i=0;i<5;i++){
        for(let j=0;j<5;j++){
            let sw=[bounds.getSouth()+i*gridStepLat, bounds.getWest()+j*gridStepLng];
            let ne=[bounds.getSouth()+(i+1)*gridStepLat, bounds.getWest()+(j+1)*gridStepLng];
            let cellPoly=turf.polygon([[
                [sw[1],sw[0]],
                [ne[1],sw[0]],
                [ne[1],ne[0]],
                [sw[1],ne[0]],
                [sw[1],sw[0]]
            ]]);
            let area=turf.area(cellPoly);
            let volumeExcavation=area*depth;
            let volumeFill=area*fillHeight;

            // مركز الخلية
            let centerLat=(sw[0]+ne[0])/2;
            let centerLng=(sw[1]+ne[1])/2;
            // رسم الخلية
            let cellLayer=L.polygon([
                [sw[0],sw[1]],
                [sw[0],ne[1]],
                [ne[0],ne[1]],
                [ne[0],sw[1]]
            ],{color:'#00ffff', fillOpacity:0.1}).addTo(gridLayer);
            gridLayer.addTo(map);

            cellCount++;
            tableHTML+=`<tr><td>${cellCount}</td><td>${centerLat.toFixed(6)}, ${centerLng.toFixed(6)}</td>
            <td>${area.toFixed(2)}</td><td>${volumeExcavation.toFixed(2)}</td><td>${volumeFill.toFixed(2)}</td></tr>`;
        }
    }
    tableHTML+=`</table>`;
    document.getElementById('table-container').innerHTML=tableHTML;

    // رسم الشكل التقريبي بعد التنفيذ
    polygonLayer.setStyle({color:'lime', fillOpacity:0.2});
    map.fitBounds(polygonLayer.getBounds());
}

// تحديث جدول النقاط
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
