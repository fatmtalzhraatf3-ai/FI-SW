<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>نظام مراقبة توليد الطاقة</title>

<style>
body {
    font-family: Arial, sans-serif;
    direction: rtl;
    margin: 0;
    background: #f5f5f5;
}

/* الهيدر */
.header {
    background: #2c3e50;
    color: white;
    padding: 15px;
    text-align: center;
}

/* الحاوية */
.container {
    padding: 20px;
}

/* الأزرار */
button {
    padding: 10px 15px;
    margin: 5px;
    border: none;
    background: #3498db;
    color: white;
    border-radius: 5px;
    cursor: pointer;
    transition: 0.3s;
}

button:hover {
    background: #1f6fa5;
}

/* الأقسام */
.section {
    margin-top: 20px;
}

/* الكروت */
.cards {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.card {
    background: white;
    padding: 15px;
    border-radius: 10px;
    flex: 1;
    min-width: 200px;
    text-align: center;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

/* الخريطة */
.map {
    width: 100%;
    height: 400px;
    background: url('https://upload.wikimedia.org/wikipedia/commons/6/6f/Map_placeholder.png');
    background-size: cover;
    border-radius: 10px;
}
</style>
</head>

<body>

<!-- الهيدر -->
<div class="header">
    <h2>جامعة قنا - كلية الآداب</h2>
    <p>نظام مراقبة توليد الطاقة من أرضية الملعب</p>
</div>

<div class="container">

    <!-- أزرار التنقل -->
    <div>
        <button onclick="showSection('map')">الخريطة</button>
        <button onclick="showSection('analysis')">التحليل</button>
        <button onclick="showSection('stats')">الإحصائيات</button>
    </div>

    <!-- الخريطة -->
    <div id="map" class="section">
        <h3>🗺️ خريطة توزيع الطاقة</h3>
        <div class="map"></div>
    </div>

    <!-- التحليل -->
    <div id="analysis" class="section" style="display:none;">
        <h3>📊 تحليل الطاقة</h3>
        <div class="cards">
            <div class="card">إجمالي الطاقة<br><b>34.75 kWh</b></div>
            <div class="card">عدد الخطوات<br><b>118,650</b></div>
            <div class="card">متوسط الطاقة<br><b>0.293 Wh</b></div>
        </div>
    </div>

    <!-- الإحصائيات -->
    <div id="stats" class="section" style="display:none;">
        <h3>📈 أعلى المناطق إنتاجًا</h3>
        <div class="cards">
            <div class="card">المنطقة الوسطى<br><b>8.6 kWh</b></div>
            <div class="card">الشمال<br><b>7.3 kWh</b></div>
            <div class="card">الجنوب<br><b>6.1 kWh</b></div>
        </div>
    </div>

</div>

<script>
// تبديل الأقسام
function showSection(id) {
    let sections = document.getElementsByClassName('section');
    
    for (let i = 0; i < sections.length; i++) {
        sections[i].style.display = 'none';
    }

    document.getElementById(id).style.display = 'block';
}
</script>

</body>
</html>
