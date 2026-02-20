<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي</title>
<style>
body { font-family: Arial, sans-serif; background:#f2f3f5; margin:0; padding:0;}
header { background:#1f3a93; color:white; padding:20px; font-size:28px; text-align:center;}
section { padding:30px; max-width:1000px; margin:auto; }
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

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY"></script>
<script>
let map, polygon, gridRectangles = [];

function generateProject(){
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const area = parseFloat(document.getElementById('area').value);
    const gridCount = parseInt(document.getElementById('gridCount').value);
    const projectType = document.getElementById('projectType').value;

    if(!lat || !lng || !area || !gridCount){ alert("من فضلك املأ جميع البيانات"); return; }

    // إزالة أي رسومات قديمة
    if(polygon) polygon.setMap(null);
    gridRectangles.forEach(rect=>rect.setMap(null));
    gridRectangles=[];

    // إنشاء الخريطة إذا لم تنشأ
    if(!map){
        map = new google.maps.Map(document.getElementById('map'),{
            center:{lat:lat,lng:lng},
            zoom:16
        });
    } else {
        map.setCenter({lat:lat,lng:lng});
    }

    // رسم Polygon افتراضي (مربع تقريبي)
    const side = Math.sqrt(area)/111000; // تقريب 1° ~ 111 km
    const bounds=[
        {lat:lat-side/2,lng:lng-side/2},
        {lat:lat-side/2,lng:lng+side/2},
        {lat:lat+side/2,lng:lng+side/2},
        {lat:lat+side/2,lng:lng-side/2},
    ];

    polygon = new google.maps.Polygon({
        paths: bounds,
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.2
    });
    polygon.setMap(map);

    // إنشاء شبكة Grid داخل Polygon
    const latStep = (bounds[2].lat - bounds[0].lat)/gridCount;
    const lngStep = (bounds[2].lng - bounds[0].lng)/gridCount;
    let gridData = [];

    for(let i=0;i<gridCount;i++){
        for(let j=0;j<gridCount;j++){
            const rectBounds=[
                {lat:bounds[0].lat+i*latStep,lng:bounds[0].lng+j*lngStep},
                {lat:bounds[0].lat+(i+1)*latStep,lng:bounds[0].lng+j*lngStep},
                {lat:bounds[0].lat+(i+1)*latStep,lng:bounds[0].lng+(j+1)*lngStep},
                {lat:bounds[0].lat+i*latStep,lng:bounds[0].lng+(j+1)*lngStep},
            ];
            const rect = new google.maps.Polygon({
                paths:rectBounds,
                strokeColor:"#008000",
                strokeOpacity:0.6,
                strokeWeight:1,
                fillColor:"#008000",
                fillOpacity:0.1
            });
            rect.setMap(map);
            gridRectangles.push(rect);

            gridData.push({
                corner1_lat: rectBounds[0].lat, corner1_lng: rectBounds[0].lng,
                corner2_lat: rectBounds[1].lat, corner2_lng: rectBounds[1].lng,
                corner3_lat: rectBounds[2].lat, corner3_lng: rectBounds[2].lng,
                corner4_lat: rectBounds[3].lat, corner4_lng: rectBounds[3].lng,
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
