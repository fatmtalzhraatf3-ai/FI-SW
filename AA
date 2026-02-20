<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي - احترافي</title>
<style>
body { font-family: Arial, sans-serif; background:#f2f3f5; margin:0; padding:0;}
header { background:#1f3a93; color:white; padding:20px; font-size:28px; text-align:center;}
section { padding:30px; max-width:900px; margin:auto; }
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
<header>المشروع المساحي الذكي - احترافي</header>
<section>
<label>اكتب إحداثيات الموقع (Latitude, Longitude):</label>
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
let map, marker;

function initMap(lat=26.1648, lng=32.7168) {
    const center = { lat: parseFloat(lat), lng: parseFloat(lng) };
    map = new google.maps.Map(document.getElementById("map"), {
        zoom: 18,
        center: center,
        mapTypeId: 'satellite'
    });
    marker = new google.maps.Marker({position: center, map: map, title:"الموقع المحدد"});
}

initMap();

function generateProject() {
    const lat = document.getElementById('lat').value;
    const lng = document.getElementById('lng').value;
    const area = parseFloat(document.getElementById('area').value);

    if(!lat || !lng || !area) { alert("من فضلك املأ جميع البيانات"); return; }

    // تحديث الخريطة والموقع
    const position = {lat: parseFloat(lat), lng: parseFloat(lng)};
    map.setCenter(position);
    marker.setPosition(position);

    // حساب Cut & Fill تقديري
    const cutfill = Math.round(area * 0.2);

    // عرض النتائج
    document.getElementById('output').innerHTML = `
        النتائج:
        <br>المساحة: ${area} م²
        <br>تقدير الحفر والردم: ${cutfill} م³
        <br>تم تحديد الموقع بدقة على الخريطة.
    `;

    // تمكين رابط التنزيل
    generateExcel(area, cutfill);
    document.getElementById('downloadLink').style.display = 'block';

    // رسم 3D طبيعي
    render3D(area);
}

// توليد ملف Excel ديناميكي للتحميل
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
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth/container.clientHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    // أرضية طبيعية
    const geometry = new THREE.PlaneGeometry(Math.sqrt(area), Math.sqrt(area));
    const material = new THREE.MeshPhongMaterial({color:0x7cfc00, side:THREE.DoubleSide});
    const plane = new THREE.Mesh(geometry, material);
    plane.rotation.x = Math.PI/2;
    scene.add(plane);

    // ملعب أو مبنى
    const fieldGeom = new THREE.BoxGeometry(Math.sqrt(area)*0.8, 2, Math.sqrt(area)*0.5);
    const fieldMat = new THREE.MeshPhongMaterial({color:0x228B22});
    const field = new THREE.Mesh(fieldGeom, fieldMat);
    field.position.y = 1;
    scene.add(field);

    // إضاءة
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10,20,10);
    scene.add(light);

    camera.position.set(0,50,50);
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
