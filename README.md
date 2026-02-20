<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مشروعي على الخريطة</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
    <style>
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background-color: #111;
            color: #fff;
        }
        header {
            padding: 15px;
            text-align: center;
            background-color: #222;
        }
        #controls {
            padding: 10px;
            text-align: center;
            background-color: #222;
        }
        input[type=text] {
            padding: 8px;
            margin: 5px;
            width: 120px;
            border-radius: 4px;
            border: none;
        }
        button {
            padding: 8px 12px;
            margin: 5px;
            border: none;
            border-radius: 4px;
            background-color: #0a84ff;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background-color: #0066cc;
        }
        #map { 
            height: 80vh; 
            width: 100%; 
        }
    </style>
</head>
<body>
    <header>
        <h1>مشروعي الجغرافي</h1>
    </header>

    <div id="controls">
        <input type="text" id="lat" placeholder="خط العرض">
        <input type="text" id="lng" placeholder="خط الطول">
        <button onclick="addMarker()">عرض المكان</button>
        <button onclick="drawGrid()">عرض الشبكة</button>
        <button onclick="drawTraverse()">رسم الترافيرس</button>
        <button onclick="draw2DShape()">رسم شكل 2D</button>
    </div>

    <div id="map"></div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script>
        // إنشاء الخريطة
        var map = L.map('map').setView([30.0444, 31.2357], 13); // نقطة مبدئية

        // خريطة OpenStreetMap
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors'
        }).addTo(map);

        var markers = []; // لتخزين الماركرات
        var traverseLine;
        var gridLayer;
        var shapeLayer;

        // إضافة ماركر
        function addMarker() {
            var lat = parseFloat(document.getElementById('lat').value);
            var lng = parseFloat(document.getElementById('lng').value);
            if(!isNaN(lat) && !isNaN(lng)){
                var marker = L.marker([lat, lng]).addTo(map).bindPopup("المكان المطلوب").openPopup();
                markers.push([lat,lng]);
                map.setView([lat, lng], 16);
            } else {
                alert("ادخلي إحداثيات صحيحة");
            }
        }

        // رسم شبكة (Grid)
        function drawGrid(){
            if(gridLayer) map.removeLayer(gridLayer);
            gridLayer = L.layerGroup();
            var bounds = map.getBounds();
            var step = 0.0015; // حجم الشبكة
            for(var lat=bounds.getSouth(); lat<bounds.getNorth(); lat+=step){
                L.polyline([[lat, bounds.getWest()], [lat, bounds.getEast()]], {color:'#555', weight:1}).addTo(gridLayer);
            }
            for(var lng=bounds.getWest(); lng<bounds.getEast(); lng+=step){
                L.polyline([[bounds.getSouth(), lng], [bounds.getNorth(), lng]], {color:'#555', weight:1}).addTo(gridLayer);
            }
            gridLayer.addTo(map);
        }

        // رسم الترافيرس تلقائي
        function drawTraverse(){
            if(traverseLine) map.removeLayer(traverseLine);
            if(markers.length<2){
                alert("يجب إدخال نقطتين على الأقل");
                return;
            }
            traverseLine = L.polyline(markers, {color:'red'}).addTo(map);
        }

        // رسم شكل 2D (مثلاً ملعب)
        function draw2DShape(){
            if(shapeLayer) map.removeLayer(shapeLayer);
            // مثال: ملعب مستطيل
            var shape = [
                [30.0444, 31.2357],
                [30.0444, 31.2367],
                [30.0454, 31.2367],
                [30.0454, 31.2357]
            ];
            shapeLayer = L.polygon(shape, {color:'green', fillOpacity:0.5}).addTo(map).bindPopup("ملعب 2D");
            map.fitBounds(shapeLayer.getBounds());
        }
    </script>
</body>
</html>
