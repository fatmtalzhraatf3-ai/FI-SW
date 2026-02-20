<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مشروعي الجغرافي المتقدم</title>

    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

    <!-- Bootstrap CSS (اختياري للتنسيق) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Turf.js لحساب المساحة والمسافات -->
    <script src="https://cdn.jsdelivr.net/npm/@turf/turf@6/turf.min.js"></script>

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
            background-color: #222;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 5px;
        }
        input[type=text], input[type=number] {
            padding: 6px;
            border-radius: 4px;
            border: none;
            width: 120px;
        }
        button {
            padding: 6px 12px;
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
            height: 60vh; 
            width: 100%; 
        }
        table {
            margin: 10px auto;
            border-collapse: collapse;
            width: 90%;
            color: #fff;
        }
        th, td {
            border: 1px solid #555;
            padding: 6px;
            text-align: center;
        }
        th {
            background-color: #333;
        }
        td {
            background-color: #222;
        }
    </style>
</head>
<body>

<header>
    <h1>مشروعي الجغرافي المتقدم</h1>
</header>

<div id="controls">
    <input type="text" id="lat" placeholder="خط العرض">
    <input type="text" id="lng" placeholder="خط الطول">
    <input type="number" id="depth" placeholder="عمق الحفر (م)" value="0.5" step="0.1">
    <button onclick="addMarker()">أضف نقطة</button>
    <button onclick="drawTraverse()">رسم الترافيرس</button>
    <button onclick="drawPolygon()">رسم المنطقة</button>
    <button onclick="calculateVolume()">احسب الحفر/الردم</button>
</div>

<div id="map"></div>

<div id="table-container"></div>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
    // إنشاء الخريطة
    var map = L.map('map').setView([30.0444, 31.2357], 15);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map);

    var markers = [];
    var traverseLine;
    var polygonLayer;

    // إضافة ماركر
    function addMarker() {
        var lat = parseFloat(document.getElementById('lat').value);
        var lng = parseFloat(document.getElementById('lng').value);
        if(isNaN(lat) || isNaN(lng)){
            alert("ادخلي إحداثيات صحيحة!");
            return;
        }
        var marker = L.marker([lat, lng]).addTo(map).bindPopup("نقطة جديدة").openPopup();
        markers.push([lat, lng]);
        map.setView([lat, lng], 16);
        updateTable();
    }

    // رسم الترافيرس
    function drawTraverse() {
        if(markers.length < 2){
            alert("يجب إدخال نقطتين على الأقل");
            return;
        }
        if(traverseLine) map.removeLayer(traverseLine);
        traverseLine = L.polyline(markers, {color: 'red'}).addTo(map);

        // حساب الطول الكلي
        let totalDistance = 0;
        for(let i=0; i<markers.length-1; i++){
            totalDistance += map.distance(markers[i], markers[i+1]);
        }
        alert("الطول الكلي للترافيرس: " + totalDistance.toFixed(2) + " متر");
    }

    // رسم Polygon
    function drawPolygon() {
        if(markers.length < 3){
            alert("يجب إدخال ثلاث نقاط على الأقل لرسم المنطقة");
            return;
        }
        if(polygonLayer) map.removeLayer(polygonLayer);
        polygonLayer = L.polygon(markers, {color:'green', fillOpacity:0.4}).addTo(map).bindPopup("المنطقة المختارة");
        map.fitBounds(polygonLayer.getBounds());
    }

    // حساب الحفر/الردم
    function calculateVolume() {
        if(!polygonLayer){
            alert("ارسم المنطقة أولا!");
            return;
        }
        var depth = parseFloat(document.getElementById('depth').value);
        var coords = polygonLayer.getLatLngs()[0].map(p => [p.lng, p.lat]); // تحويل للـ [lng,lat]
        coords.push(coords[0]); // إغلاق الشكل
        var polygon = turf.polygon([coords]);
        var area = turf.area(polygon);
