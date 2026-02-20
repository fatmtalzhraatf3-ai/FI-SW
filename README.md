<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-sA+g0IVHkNbdj5N3w8u3Fjae0S4p6lce1x9e+WkCDjo=" crossorigin=""/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-qbWJs1hCuq3P3Fmj6UVX8gErh6k2hfs7iQbFmXrS1KQ=" crossorigin=""></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<style>
body { font-family: Arial, sans-serif; background:#f2f3f5; margin:0; padding:0;}
header { background:#1f3a93; color:white; padding:20px; font-size:28px; text-align:center;}
section { padding:20px; max-width:1000px; margin:auto; }
label { display:block; margin-top:15px; font-weight:bold; }
input, select, button { padding:10px; margin:10px 0; font-size:16px; width:100%; box-sizing:border-box; }
button { background:#1f3a93; color:white; border:none; cursor:pointer; transition:0.3s; }
button:hover { background:#163570; }
#map { width:100%; height:600px; border:2px solid #333; margin-top:20px; }
#output { margin-top:20px; font-size:18px; font-weight:bold; }
#downloadLink { display:none; margin-top:15px; padding:10px; background:#27ae60; color:white; text-decoration:none; font-weight:bold; }
#downloadLink:hover { background:#1e8449; }
</style>
</head>
<body>
<header>المشروع المساحي الذكي</header>
<section>
<label>خط العرض (Latitude):</label>
<input type="text" id="lat" placeholder="مثال: 26.1648">
<label>خط الطول (Longitude):</label>
<input type="text" id="lng" placeholder="مثال: 32.7168">
<label>المساحة بالمتر²:</label>
<input type="number" id="area" placeholder="مثال: 5000">
<label>نوع المشروع:</label>
<select id="projectType">
  <option value="ملعب">ملعب</option>
  <option value="مبنى">مبنى</option>
</select>
<label>عدد المقاطع بالشبكة:</label>
<input type="number" id="gridCount" placeholder="مثال: 5">
<button onclick="generateProject()">عرض المشروع</button>

<div id="map"></div>
<div id="output"></div>
<a id="downloadLink" href="#" download="project_grid.xlsx">تنزيل ملف الشبكة Excel</a>
</section>

<script>
let map, polygonLayer, gridLayers=[];

function generateProject(){
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const area = parseFloat(document.getElementById('area').value);
    const gridCount = parseInt(document.getElementById('gridCount').value);
    const projectType = document.getElementById('projectType').value;

    if(!lat || !lng || !area || !gridCount){ alert("من فضلك املأ جميع البيانات"); return; }

    // إزالة أي رسومات قديمة
    if(polygonLayer) map.removeLayer(polygonLayer);
    gridLayers.forEach(l=>map.removeLayer(l));
    gridLayers=[];

    // إنشاء الخريطة إذا لم تنشأ
    if(!map){
        map = L.map('map').setView([lat, lng], 16);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
            attribution:'© OpenStreetMap'
        }).addTo(map);
    } else {
        map.setView([lat,lng],16);
    }

    // رسم Polygon افتراضي (مربع تقريبي)
    const side = Math.sqrt(area)/111000; // تقريب 1° ~ 111 km
    const bounds=[
        [lat-side/2,lng-side/2],
        [lat-side/2,lng+side/2],
        [lat+side/2,lng+side/2],
        [lat+side/2,lng-side/2]
    ];
    polygonLayer = L.polygon(bounds,{color:'red', fillOpacity:0.2}).addTo(map);

    // إنشاء شبكة Grid داخل Polygon
    const latStep = (bounds[2][0]-bounds[0][0])/gridCount;
    const lngStep = (bounds[2][1]-bounds[0][1])/gridCount;
    let gridData = [];

    for(let i=0;i<gridCount;i++){
        for(let j=0;j<gridCount;j++){
            const rect=[
                [bounds[0][0]+i*latStep,bounds[0][1]+j*lngStep],
                [bounds[0][0]+(i+1)*latStep,bounds[0][1]+j*lngStep],
                [bounds[0][0]+(i+1)*latStep,bounds[0][1]+(j+1)*lngStep],
                [bounds[0][0]+i*latStep,bounds[0][1]+(j+1)*lngStep]
            ];
            const rectLayer=L.polygon(rect,{color:'green', fillOpacity:0.1}).addTo(map);
            gridLayers.push(rectLayer);

            gridData.push({
                corner1_lat: rect[0][0], corner1_lng: rect[0][1],
                corner2_lat: rect[1][0], corner2_lng: rect[1][1],
                corner3_lat: rect[2][0], corner3_lng: rect[2][1],
                corner4_lat: rect[3][0], corner4_lng: rect[3][1],
                projectType: projectType
            });
        }
    }

    document.getElementById('output').innerText=`تم إنشاء المشروع: ${projectType} وعدد المربعات: ${gridCount*gridCount}`;

    // تجهيز زر التنزيل بعد العرض
    const ws = XLSX.utils.json_to_sheet(gridData);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb,ws,"Grid");
    const wbout = XLSX.write(wb,{bookType:'xlsx', type:'array'});
    const blob = new Blob([wbout],{type:"application/octet-stream"});
    const link = document.getElementById('downloadLink');
    link.href = URL.createObjectURL(blob);
    link.style.display='block';
}
</script>
</body>
</html>
