<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>المشروع المساحي الذكي - نسخة مطورة</title>
<style>
body { font-family: Arial, sans-serif; text-align:center; background:#e8f5e9; margin:0; padding:0; }
h1 { margin-top:20px; color:#1b5e20; }
input, button { margin:10px; padding:10px; font-size:16px; border-radius:5px; border:1px solid #1b5e20; }
#map, #terrain3D { width: 90%; height: 400px; margin: 20px auto; border:2px solid #333; }
#downloadLink { display:block; margin-top:15px; font-weight:bold; color:#1b5e20; }
</style>
</head>
<body>

<h1>المشروع المساحي الذكي - نسخة مطورة</h1>

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
<script>
function generateProject() {
    const lat = parseFloat(document.getElementById('lat').value);
    const lng = parseFloat(document.getElementById('lng').value);
    const length = parseFloat(document.getElementById('length').value);
    const width = parseFloat(document.getElementById('width').value);
    if(!lat || !lng || !length || !width){ alert("ادخل كل البيانات"); return; }

    // توليد شبكة نقاط الأرض
    const rows = Math.ceil(length / 5);
    const cols = Math.ceil(width / 5);
    const points = [];
    for(let i=0;i<=rows;i++){
        for(let j=0;j<=cols;j++){
            // ارتفاع طبيعي للأرض + تدرج عشوائي صغير
            points.push({
                x: j*5,
                y: i*5,
                elevation: 100 + Math.random()*2
            });
        }
    }

    // اضافة ملعب او مبنى وسط الأرض
    const centerRow = Math.floor(rows/2);
    const centerCol = Math.floor(cols/2);
    const buildingRows = Math.min(10, rows);
    const buildingCols = Math.min(20, cols);

    for(let i=centerRow-Math.floor(buildingRows/2); i<=centerRow+Math.floor(buildingRows/2); i++){
        for(let j=centerCol-Math.floor(buildingCols/2); j<=centerCol+Math.floor(buildingCols/2); j++){
            const idx = i*(cols+1)+j;
            if(points[idx]) points[idx].elevation += 2; // ارتفاع المبنى/الملعب
        }
    }

    // حساب Cut & Fill
    points.forEach(p => { p.cutfill = Math.round(p.elevation - 100); });

    generateExcel(points);
    drawMap(lat, lng, points);
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

function drawMap(lat, lng, points){
    document.getElementById('map').innerHTML = "";
    const mapDiv = document.getElementById('map');
    const iframe = document.createElement('iframe');
    iframe.width="100%"; iframe.height="100%";
    iframe.src = `https://maps.google.com/maps?q=${lat},${lng}&hl=ar&z=17&output=embed`;
    mapDiv.appendChild(iframe);
}

function draw3DTerrain(points, rows, cols){
    document.getElementById('terrain3D').innerHTML = "";
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, 0.9*window.innerWidth/400, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(document.getElementById('terrain3D').clientWidth, 400);
    document.getElementById('terrain3D').appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);

    // رسم شبكة الارض
    const geometry = new THREE.PlaneGeometry(cols*5, rows*5, cols, rows);
    geometry.vertices.forEach((v, idx)=>{ 
        if(points[idx]) v.z = points[idx].elevation/2; 
    });
    const material = new THREE.MeshLambertMaterial({color:0x2e7d32, wireframe:false});
    const plane = new THREE.Mesh(geometry, material);
    plane.rotation.x = -Math.PI/2;
    scene.add(plane);

    // ضوء طبيعي
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(50,50,50);
    scene.add(light);
    const ambient = new THREE.AmbientLight(0x404040);
    scene.add(ambient);

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
