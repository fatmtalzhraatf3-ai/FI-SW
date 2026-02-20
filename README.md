<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي - Google Earth</title>
<style>
body { font-family: Arial, sans-serif; background:#f2f3f5; margin:0; padding:0;}
header { background:#1f3a93; color:white; padding:20px; font-size:28px; text-align:center;}
section { padding:30px; max-width:1000px; margin:auto; }
label { display:block; margin-top:15px; font-weight:bold; }
input, button { padding:10px; margin:10px 0; font-size:16px; width:100%; box-sizing:border-box; }
button { background:#1f3a93; color:white; border:none; cursor:pointer; transition:0.3s; }
button:hover { background:#163570; }
#map { width:100%; height:600px; border:2px solid #333; margin-top:20px; }
#output { margin-top:20px; font-size:18px; font-weight:bold; }
#downloadLink { display:none; margin-top:15px; padding:10px; background:#27ae60; color:white; text-decoration:none; font-weight:bold; }
#downloadLink:hover { background:#1e8449; }
</style>
</head>
<body>
<header>المشروع المساحي - شبكة وكروكي Google Earth</header>
<section>
<label>إحداثيات الموقع (Latitude, Longitude):</label>
<input type="text" id="lat" placeholder="مثال: 26.1648">
<input type="text" id="lng" placeholder="مثال: 32.7168">
<label>المساحة بالمتر²:</label>
<input type="number" id="area" placeholder="مثال: 5000">
<label>عدد المقاطع بالشبكة:</label>
<input type="number" id="gridCount" placeholder="مثال: 5">
<button onclick="generateProject()">عرض المشروع</button>

<div id="map"></div>
<div id="output"></div>
<a id="downloadLink" href="#" download="project_grid.xlsx">تنزيل ملف الشبكة Excel</a>
</section>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script src="https://www.gstatic.com/earth/client/api.js"></script>
<script>
let ge, placemarkPolygon, gridPolygons = [];

function initEarth() {
    google.earth.createInstance('map', initCB, failureCB);
}

function initCB(instance) {
    ge = instance;
    ge.getWindow().setVisibility(true);
    ge.getNavigationControl().setVisibility(ge.VISIBILITY_AUTO);
    ge.getLayerRoot().enableLayerById(ge.LAYER_BORDERS, true);
    ge.getLayerRoot().enableLayerById(ge.LAYER_TERRAIN, true);
}

function failureCB(errorCode) { alert("خطأ في تحميل Google Earth"); }

initEarth();

function generateProject() {
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const area = parseFloat(document.getElementById('area').value);
    const gridCount = parseInt(document.getElementById('gridCount').value);

    if(!lat || !lng || !area || !gridCount){ alert("من فضلك املأ جميع البيانات"); return; }

    // إزالة أي رسومات قديمة
    if(placemarkPolygon) ge.getFeatures().removeChild(placemarkPolygon);
    gridPolygons.forEach(p => ge.getFeatures().removeChild(p));
    gridPolygons = [];

    // رسم Polygon تقريبي للمساحة كمربع
    const side = Math.sqrt(area)/111000; // 1° ~ 111 km
    const bounds = [
        {lat: lat - side/2, lng: lng - side/2},
        {lat: lat - side/2, lng: lng + side/2},
        {lat: lat + side/2, lng: lng + side/2},
        {lat: lat + side/2, lng: lng - side/2}
    ];

    placemarkPolygon = ge.createPlacemark('');
    const poly = ge.createPolygon('');
    const outer = ge.createLinearRing('');
    bounds.forEach(pt => outer.getCoordinates().pushLatLngAlt(pt.lat, pt.lng, 0));
    outer.getCoordinates().pushLatLngAlt(bounds[0].lat, bounds[0].lng, 0);
    poly.setOuterBoundary(outer);
    poly.setAltitudeMode(ge.ALTITUDE_CLAMP_TO_GROUND);
    placemarkPolygon.setGeometry(poly);
    ge.getFeatures().appendChild(placemarkPolygon);

    // إنشاء الشبكة
    const latStep = (bounds[2].lat - bounds[0].lat)/gridCount;
    const lngStep = (bounds[2].lng - bounds[0].lng)/gridCount;
    let gridData = [];

    for(let i=0;i<gridCount;i++){
        for(let j=0;j<gridCount;j++){
            const gBounds = [
                {lat: bounds[0].lat + i*latStep, lng: bounds[0].lng + j*lngStep},
                {lat: bounds[0].lat + (i+1)*latStep, lng: bounds[0].lng + j*lngStep},
                {lat: bounds[0].lat + (i+1)*latStep, lng: bounds[0].lng + (j+1)*lngStep},
                {lat: bounds[0].lat + i*latStep, lng: bounds[0].lng + (j+1)*lngStep}
            ];

            const gp = ge.createPlacemark('');
            const gPoly = ge.createPolygon('');
            const gRing = ge.createLinearRing('');
            gBounds.forEach(pt => gRing.getCoordinates().pushLatLngAlt(pt.lat, pt.lng, 0));
            gRing.getCoordinates().pushLatLngAlt(gBounds[0].lat, gBounds[0].lng, 0);
            gPoly.setOuterBoundary(gRing);
            gPoly.setAltitudeMode(ge.ALTITUDE_CLAMP_TO_GROUND);
            gp.setGeometry(gPoly);
            ge.getFeatures().appendChild(gp);
            gridPolygons.push(gp);

            gridData.push({
                corner1_lat: gBounds[0].lat, corner1_lng: gBounds[0].lng,
                corner2_lat: gBounds[1].lat, corner2_lng: gBounds[1].lng,
                corner3_lat: gBounds[2].lat, corner3_lng: gBounds[2].lng,
                corner4_lat: gBounds[3].lat, corner4_lng: gBounds[3].lng
            });
        }
    }

    // إنشاء Excel
    const ws = XLSX.utils.json_to_sheet(gridData);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Grid");
    const wbout = XLSX.write(wb,{bookType:'xlsx', type:'array'});
    const blob = new Blob([wbout],{type:"application/octet-stream"});
    const link = document.getElementById('downloadLink');
    link.href = URL.createObjectURL(blob);
    link.style.display = 'block';
    link.innerText = "تنزيل ملف الشبكة Excel";
    document.getElementById('output').innerText = `تم إنشاء الشبكة وعدد المربعات: ${gridCount*gridCount}`;
}
</script>
</body>
</html>
