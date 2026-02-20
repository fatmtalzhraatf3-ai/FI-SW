<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي - خرائط دقيقة</title>
<style>
body { font-family: Arial, sans-serif; text-align:center; background:#e8f5e9; margin:0; padding:0; }
h1 { margin-top:20px; color:#1b5e20; }
input, button { margin:10px; padding:10px; font-size:16px; border-radius:5px; border:1px solid #1b5e20; }
#map, #terrain3D { width: 90%; height: 400px; margin: 20px auto; border:2px solid #333; }
#downloadLink { display:block; margin-top:15px; font-weight:bold; color:#1b5e20; }
</style>
</head>
<body>

<h1>المشروع المساحي الذكي - خرائط دقيقة</h1>

<label>خط العرض: </label><input type="number" id="lat" step="0.0001" placeholder="مثال: 26.1648"><br>
<label>خط الطول: </label><input type="number" id="lng" step="0.0001" placeholder="مثال: 32.7168"><br>
<label>طول الأرض (متر): </label><input type="number" id="length" placeholder="مثال: 50"><br>
<label>عرض الأرض (متر): </label><input type="number" id="width" placeholder="مثال: 30"><br>
<button onclick="generateProject()">انشاء المشروع</button>

<a id="downloadLink" style="display:none">تحميل تقرير Excel</a>

<div id="map"></div>
<div id="terrain3D"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r152/three.min.js"></script>
<script src="https://threejs.org/examples/js/controls/OrbitControls.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script async src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY"></script>

<script>
let map, polygon;

function generateProject() {
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const length = parseFloat(document.getElementById('length').value);
    const width = parseFloat(document.getElementById('width').value);
    if(!lat || !lng || !length || !width){ alert("ادخل كل البيانات"); return; }

    // إنشاء شبكة نقاط الأرض
    const rows = Math.ceil(length / 5);
    const cols = Math.ceil(width / 5);
    const points = [];
    for(let i=0;i<=rows;i++){
        for(let j=0;j<=cols;j++){
            points.push({ x: j*5, y: i*5, elevation: 100 + Math.random()*2 });
        }
    }

    // اضافة ملعب/مبنى
    const centerRow = Math.floor(rows/2);
    const centerCol = Math.floor(cols/2);
    const buildingRows = Math.min(10, rows);
    const buildingCols = Math.min(20, cols);
    for(let i=centerRow-Math.floor(buildingRows/2); i<=centerRow+Math.floor(buildingRows/2); i++){
        for(let j=centerCol-Math.floor(buildingCols/2); j<=centerCol+Math.floor(buildingCols/2); j++){
            const idx = i*(cols+1)+j;
            if(points[idx]) points[idx].elevation += 2;
        }
    }

    // حساب Cut & Fill
    points.forEach(p => { p.cutfill = Math.round(p.elevation - 100); });

    generateExcel(points);
    drawMap(lat, lng, length, width);
    draw3DTerrain(points, rows, cols);
    alert("تم إنشاء المشروع المطور بنجاح!");
}

function generateExcel(points){
    const ws = XLSX.utils.json_to_sheet(points);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Project");
    XLSX.writeFile(wb, "Project_Report.xlsx");
    document.getElementById('downloadLink').style.display = 'block';
}

function drawMap(lat, lng, length, width){
    const mapDiv = document.getElementById('map');
    mapDiv.innerHTML = "";
    map = new google.maps.Map(mapDiv, {
        center: {lat: lat, lng: lng},
        zoom: 18,
        mapTypeId: 'satellite'
    });

    // تحويل الأبعاد بالمتر إلى درجات تقريبا (تقريب بسيط)
    const latOffset = (length/111000); // طول 1 درجة ~ 111 كم
    const lngOffset = (width/(111000*Math.cos(lat*Math.PI/180)));

    const polygonCoords = [
        {lat: lat, lng: lng},
        {lat: lat + latOffset, lng: lng},
        {lat: lat + latOffset, lng: lng + lngOffset},
        {lat: lat, lng: lng + lngOffset}
    ];

    // رسم المضلع
    if(polygon) polygon.setMap(null);
    polygon = new google.maps.Polygon({
        paths: polygonCoords,
        strokeColor: '#FF0000',
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: '#FF0000',
        fillOpacity: 0.35
    });
    polygon.setMap(map);

    // مركز الكروكي
    map.fitBounds(new google.maps.LatLngBounds(
        new google.maps.LatLng(lat, lng),
        new google.maps.LatLng(lat + latOffset, lng + lngOffset)
    ));
}

function draw3DTerrain(points, rows, cols){
    const div = document.getElementById('terrain3D');
    div.innerHTML = "";
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, div.clientWidth/400, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(div.clientWidth, 400);
    div.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);

    const geometry = new THREE.PlaneGeometry(cols*5, rows*5, cols, rows);
    geometry.vertices.forEach((v, idx)=>{ 
        if(points[idx]) v.z = points[idx].elevation/2; 
    });
    const material = new THREE.MeshLambertMaterial({color:0x2e7d32, wireframe:false});
    const plane = new THREE.Mesh(geometry, material);
    plane.rotation.x = -Math.PI/2;
    scene.add(plane);

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(50,50,50);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x404040));

    camera.position.set(cols*5, rows*5, 50);
    controls.update();

    function animate(){
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
    }
    animate();
}
</script>

</body>
</html>
