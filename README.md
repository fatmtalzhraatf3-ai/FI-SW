<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي - 3D & Map</title>
<style>
body { font-family: Arial, sans-serif; background:#f2f3f5; margin:0; padding:0;}
header { background:#1f3a93; color:white; padding:20px; font-size:28px; text-align:center;}
section { padding:30px; max-width:1000px; margin:auto; }
label { display:block; margin-top:15px; font-weight:bold; }
input, button { padding:10px; margin:10px 0; font-size:16px; width:100%; box-sizing:border-box; }
button { background:#1f3a93; color:white; border:none; cursor:pointer; transition:0.3s; }
button:hover { background:#163570; }
#map { width:100%; height:400px; border:2px solid #333; margin-top:20px; }
#render3D { width:100%; height:400px; border:2px solid #333; margin-top:20px; }
#output { margin-top:20px; font-size:18px; font-weight:bold; }
#downloadLink { display:none; margin-top:15px; padding:10px; background:#27ae60; color:white; text-decoration:none; font-weight:bold; }
#downloadLink:hover { background:#1e8449; }
</style>
</head>
<body>
<header>المشروع المساحي الذكي - 3D & Map</header>
<section>
<label>إحداثيات الموقع (Latitude, Longitude):</label>
<input type="text" id="lat" placeholder="مثال: 26.1648">
<input type="text" id="lng" placeholder="مثال: 32.7168">
<label>المساحة بالمتر²:</label>
<input type="number" id="area" placeholder="مثال: 5000">
<button onclick="generateProject()">عرض المشروع</button>

<div id="map"></div>
<div id="render3D"></div>
<div id="output"></div>
<a id="downloadLink" href="#" download="project_results.xlsx">تنزيل النتائج</a>
</section>

<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.159.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>

<script>
let map, marker, polygon;

function initMap(lat=26.1648, lng=32.7168) {
    const center = { lat: parseFloat(lat), lng: parseFloat(lng) };
    map = new google.maps.Map(document.getElementById("map"), {
        zoom: 19,
        center: center,
        mapTypeId: 'satellite'
    });
    marker = new google.maps.Marker({position: center, map: map, title:"الموقع المحدد"});
}

initMap();

function generateProject() {
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const area = parseFloat(document.getElementById('area').value);

    if(!lat || !lng || !area) { alert("من فضلك املأ جميع البيانات"); return; }

    const position = {lat: lat, lng: lng};
    map.setCenter(position);
    marker.setPosition(position);

    // رسم كروكي المساحة كمضلع تقريبي مربع
    if(polygon) polygon.setMap(null);
    const side = Math.sqrt(area) / 111000; // تقريب طول الجانب بدرجة (1° ≈ 111km)
    const bounds = [
        {lat: lat - side/2, lng: lng - side/2},
        {lat: lat - side/2, lng: lng + side/2},
        {lat: lat + side/2, lng: lng + side/2},
        {lat: lat + side/2, lng: lng - side/2}
    ];
    polygon = new google.maps.Polygon({
        paths: bounds,
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35
    });
    polygon.setMap(map);

    // حساب Cut & Fill تقديري
    const cutfill = Math.round(area * 0.2);

    document.getElementById('output').innerHTML = `
        النتائج:
        <br>المساحة: ${area} م²
        <br>تقدير الحفر والردم: ${cutfill} م³
        <br>تم تحديد الموقع ورسم الكروكي على الخريطة.
    `;

    // تمكين رابط التنزيل
    generateExcel(area, cutfill);
    document.getElementById('downloadLink').style.display = 'block';

    // رسم 3D واقعي
    render3D(area);
}

// توليد ملف Excel ديناميكي
function generateExcel(area, cutfill) {
    const wb = XLSX.utils.book_new();
    const ws_data = [
        ["المساحة", "تقدير الحفر والردم"],
        [area, cutfill]
    ];
    const ws = XLSX.utils.aoa_to_sheet(ws_data);
    XLSX.utils.book_append_sheet(wb, ws, "Project");
    const wbout = XLSX.write(wb,{bookType:'xlsx', type:'binary'});
    function s2ab(s) {
        const buf = new ArrayBuffer(s.length);
        const view = new Uint8Array(buf);
        for(let i=0;i<s.length;i++) view[i] = s.charCodeAt(i) & 0xFF;
        return buf;
    }
    const blob = new Blob([s2ab(wbout)],{type:"application/octet-stream"});
    const url = URL.createObjectURL(blob);
    document.getElementById('downloadLink').href = url;
}

function render3D(area) {
    const container = document.getElementById('render3D');
    container.innerHTML = "";
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xa0d0ff);
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth/container.clientHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    // أرضية طبيعية
    const geometry = new THREE.PlaneGeometry(Math.sqrt(area), Math.sqrt(area));
    const material = new THREE.MeshPhongMaterial({color:0x228B22, side:THREE.DoubleSide});
    const plane = new THREE.Mesh(geometry, material);
    plane.rotation.x = Math.PI/2;
    scene.add(plane);

    // ملعب / مبنى طبيعي
    const fieldGeom = new THREE.BoxGeometry(Math.sqrt(area)*0.8, 2, Math.sqrt(area)*0.5);
    const fieldMat = new THREE.MeshPhongMaterial({color:0x556B2F});
    const field = new THREE.Mesh(fieldGeom, fieldMat);
    field.position.y = 1;
    scene.add(field);

    // إضاءة طبيعية
    const ambientLight = new THREE.AmbientLight(0xffffff,0.6);
    scene.add(ambientLight);
    const dirLight = new THREE.DirectionalLight(0xffffff,1);
    dirLight.position.set(20,50,20);
    scene.add(dirLight);

    camera.position.set(Math.sqrt(area)/2,50,Math.sqrt(area));
    camera.lookAt(0,0,0);

    const animate = function() {
        requestAnimationFrame(animate);
        field.rotation.y += 0.001;
        renderer.render(scene, camera);
    };
    animate();
}
</script>
</body>
</html>
