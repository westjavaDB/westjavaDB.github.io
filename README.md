<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>West Java Workforce & Industry — Interactive Map</title>

  <!-- Leaflet CSS & JS -->
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
  />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <!-- Simple styling (Bootstrap-like minimal) -->
  <style>
    html,body{height:100%;margin:0;font-family:Arial, Helvetica, sans-serif}
    #map{position:absolute;top:0;bottom:0;left:0;right:320px;}
    #sidebar{position:absolute;top:0;right:0;width:320px;height:100%;background:#fff;border-left:1px solid #ddd;overflow:auto;padding:12px;box-sizing:border-box;}
    h3{margin-top:0}
    .legend {position: fixed; bottom: 140px; left: 16px; z-index:9999; background:white; padding:8px; border:1px solid #888; font-size:13px; max-width:260px;}
    .legend .item {margin:6px 0;display:flex;align-items:center}
    .legend .swatch {width:18px;height:12px;margin-right:8px;border:1px solid #ccc}
    table {width:100%;border-collapse:collapse;font-size:13px}
    th,td{padding:6px;border-bottom:1px solid #eee;text-align:left}
    input[type="search"]{width:100%;padding:6px;margin-bottom:8px;box-sizing:border-box;}
    .footer-note{font-size:12px;color:#666;margin-top:8px}
  </style>
</head>
<body>

  <div id="map"></div>

  <div id="sidebar">
    <h3>Workforce & Industry — West Java (2024)</h3>
    <input id="searchBox" type="search" placeholder="Search region / city... (type and press Enter)" />

    <div id="tableWrap" style="max-height:58vh;overflow:auto">
      <table id="dataTable">
        <thead>
          <tr><th>Region/City</th><th>Companies</th><th>Workforce</th><th>Unemployment %</th></tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <div class="footer-note">
      Markers sized by workforce estimate. Colors match dominant sector groups.
      <br><br>
      Ready to host: upload this file to GitHub Pages / Netlify.
    </div>
  </div>

  <div class="legend" id="map-legend">
    <strong>Legend - SMK Top1</strong>
    <div class="item"><span class="swatch" style="background:#e41a1c"></span>Agribisnis & Agroteknologi</div>
    <div class="item"><span class="swatch" style="background:#449c76"></span>Bisnis & Manajemen</div>
    <div class="item"><span class="swatch" style="background:#ad5882"></span>Seni & Ekonomi Kreatif</div>
    <div class="item"><span class="swatch" style="background:#ffe628"></span>Teknologi Informasi & Komunikasi</div>
    <div class="item"><span class="swatch" style="background:#c76764"></span>Teknologi Manufaktur & Rekayasa</div>
  </div>

  <!-- Embedded CSV data (from your file) -->
  <script id="industrial-csv" type="text/plain">
Region/City,Number of Industrial Companies (2024),Workforce (Formal+Informal 2024),Percentage of Total Workforce,Dominant Industrial Sectors & Notes,Unemployment Rate (%)
Kota Cimahi,200-300,150000,0.5%,"Manufacturing, trade, services, high urban employment",8.97
Kabupaten Bekasi,1000+,2800000,10.7%,"Heavy industry, electronics, logistics hubs, largest formal sector",8.82
Kota Sukabumi,150-250,160000,0.6%,"Small to medium industries, agro-processing",8.34
Kota Bogor,250-350,1300000,4.9%,"Manufacturing, tourism, services",8.13
Kabupaten Karawang,500-700,2200000,8.4%,"Automotive, electronics, production zones",8.04
Kota Bekasi,800-1000,1800000,6.9%,"Logistics, manufacturing, trade",7.82
Kota Bandung,400-600,2100000,8.0%,"Creative industries, tech, manufacturing",7.4
Kabupaten Bogor,300-400,2500000,9.4%,"Agro industry, manufacturing, urban-industrial zone",7.32
Kabupaten Sukabumi,150-250,400000,1.5%,"Agriculture, small manufacturing, rural industries",7.32
Kabupaten Bandung Barat,200-300,600000,2.3%,"Textile, small manufacturing, growing industries",7.1
Kabupaten Sumedang,100-200,300000,1.1%,"Agro-industries, small manufacturing",6.9
Kota Tasikmalaya,150-250,350000,1.3%,"Textile, small industries, local crafts",6.55
Kabupaten Bandung,250-350,3000000,11.5%,"Manufacturing, agro-industry, trade",6.5
Kota Depok,300-400,2000000,7.7%,"Services, retail, manufacturing",6.3
Kota Cirebon,200-300,167000,0.6%,"Food processing, furniture, logistics",6.29
Kabupaten Subang,150-250,500000,1.9%,"Agro-industry, manufacturing",6.0
Kabupaten Purwakarta,200-300,600000,2.3%,"Manufacturing, electronics",5.8
Kabupaten Tasikmalaya,150-250,450000,1.7%,"Textile, agro-industry",5.5
Kota Banjar,100-200,200000,0.8%,"Small industries, crafts",5.44
Kabupaten Cirebon,200-300,1000000,3.8%,"Food/beverage, furniture, manufacturing",5.2
Kabupaten Majalengka,100-200,350000,1.3%,"Agriculture-based industries",4.9
Kabupaten Indramayu,150-250,600000,2.3%,"Oil refining, agro-industry",4.7
Kabupaten Kuningan,100-200,300000,1.1%,"Agriculture, small manufacturing",4.5
Kabupaten Garut,150-250,400000,1.5%,"Textile, agro-industry",4.4
Kabupaten Cianjur,200-300,1200000,4.6%,"Agriculture, manufacturing, trade",4.2
Kabupaten Pangandaran,Low,Low,Mostly informal,"Tourism, agriculture, small manufacturing",4.0
Kabupaten Ciamis,Low,Low,Mostly informal,"Agriculture, small-scale manufacturing",3.37
  </script>

  <script>
    // Small lookup of approximate coordinates for main regions/cities in the CSV.
    // These are approximate centroids for visualization only.
    const coordsLookup = {
      "Kota Cimahi":[-6.8797,107.5469],
      "Kabupaten Bekasi":[-6.3090,107.0219],
      "Kota Sukabumi":[-6.9255,106.9254],
      "Kota Bogor":[-6.5975,106.8069],
      "Kabupaten Karawang":[-6.3071,107.3170],
      "Kota Bekasi":[-6.2349,107.0011],
      "Kota Bandung":[-6.9175,107.6191],
      "Kabupaten Bogor":[-6.5946,106.7914],
      "Kabupaten Sukabumi":[-6.8077,106.9280],
      "Kabupaten Bandung Barat":[-6.8167,107.5315],
      "Kabupaten Sumedang":[-6.8168,107.9180],
      "Kota Tasikmalaya":[-7.3273,108.2227],
      "Kabupaten Bandung":[-7.0254,107.7481],
      "Kota Depok":[-6.4025,106.7949],
      "Kota Cirebon":[-6.7320,108.5520],
      "Kabupaten Subang":[-6.5511,107.5559],
      "Kabupaten Purwakarta":[-6.5581,107.4452],
      "Kabupaten Tasikmalaya":[-7.3188,108.2103],
      "Kota Banjar":[-7.3341,108.5519],
      "Kabupaten Cirebon":[-6.8000,108.5500],
      "Kabupaten Majalengka":[-6.8497,108.2190],
      "Kabupaten Indramayu":[-6.3271,108.3297],
      "Kabupaten Kuningan":[-6.9768,108.4735],
      "Kabupaten Garut":[-7.2146,107.9156],
      "Kabupaten Cianjur":[-6.8182,107.1632],
      "Kabupaten Pangandaran":[-7.6469,108.4993],
      "Kabupaten Ciamis":[-7.3184,108.3490]
    };

    // Sector color mapping (matching legend)
    function sectorColor(text){
      text = (text||"").toLowerCase();
      if (text.includes("agribisnis") || text.includes("agro")) return "#e41a1c";
      if (text.includes("bisnis") || text.includes("management") || text.includes("manajemen")) return "#449c76";
      if (text.includes("seni") || text.includes("kreatif") || text.includes("creative")) return "#ad5882";
      if (text.includes("teknologi informasi") || text.includes("it") || text.includes("telekom") || text.includes("technology")) return "#ffe628";
      if (text.includes("manufaktur") || text.includes("manufacturing") || text.includes("automotive") || text.includes("electronics")) return "#c76764";
      // fallback
      return "#6a737d";
    }

    // Parse CSV (simple)
    function parseCSV(csvText){
      const lines = csvText.trim().split(/\r?\n/).map(r=>r.trim()).filter(Boolean);
      const header = lines.shift().split(",").map(h=>h.trim());
      const rows = lines.map(line=>{
        // handle quoted commas: naive approach using regex to split on commas not inside quotes
        const values = line.match(/(".*?"|[^",\s]+)(?=\s*,|\s*$)/g) || [];
        const cleaned = values.map(v=>v.replace(/^"|"$/g,"").trim());
        const obj = {};
        header.forEach((h,i)=> obj[h]=cleaned[i]||"");
        return obj;
      });
      return rows;
    }

    // Try to convert workforce string to number for sizing. Accept "Low" or ranges.
    function parseWorkforce(w){
      if (!w) return null;
      w = w.toString().replace(/\s/g,"");
      if (w.toLowerCase()==="low") return 20000; // placeholder small
      // range e.g. 150000 or "2,800,000" or "200-300" companies not workforce
      const n = parseInt(w.replace(/[^0-9]/g,""),10);
      if (!isNaN(n) && n>0) return n;
      return null;
    }

    // Build map
    const map = L.map('map', {center: [-6.85, 107.5], zoom: 8});
    L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png',{
      maxZoom: 19, attribution:'&copy; OpenStreetMap & CartoDB'
    }).addTo(map);

    // read CSV text
    const csvText = document.getElementById('industrial-csv').textContent;
    const dataRows = parseCSV(csvText);

    // populate table and markers
    const tbody = document.querySelector('#dataTable tbody');
    const markersGroup = L.layerGroup().addTo(map);

    dataRows.forEach(row=>{
      const name = row["Region/City"] || row["Region/City".trim()];
      const workforceRaw = row["Workforce (Formal+Informal 2024)"] || row["Workforce (Formal+Informal 2024)".trim()] || row["Workforce (Formal+Informal 2024)"];
      const workforce = parseWorkforce(workforceRaw) || null;
      const companies = row["Number of Industrial Companies (2024)"] || "";
      const unemployment = row["Unemployment Rate (%)"] || "";
      const sector = row["Dominant Industrial Sectors & Notes"] || "";

      // add to table
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${name}</td><td>${companies}</td><td>${workforceRaw||""}</td><td>${unemployment}</td>`;
      tbody.appendChild(tr);

      // marker if coords available
      if (coordsLookup[name]){
        const latlng = coordsLookup[name];
        // marker color by sector
        const color = sectorColor(sector);
        // size mapping
        const radius = Math.max(6, Math.min(60, (workforce ? Math.sqrt(workforce)/50 : 8)));
        const circle = L.circleMarker(latlng, {
          radius: radius,
          fillColor: color,
          color: "#222",
          weight: 0.8,
          opacity: 1,
          fillOpacity: 0.8
        }).bindPopup(`<strong>${name}</strong><br/>Companies: ${companies}<br/>Workforce: ${workforceRaw || "N/A"}<br/>Unemployment: ${unemployment}<br/>Sectors: ${sector}`);
        circle.addTo(markersGroup);
      }
    });

    // Table row click -> pan to marker if exists
    document.querySelectorAll('#dataTable tbody tr').forEach(tr=>{
      tr.style.cursor = "pointer";
      tr.addEventListener('click', ()=>{
        const name = tr.cells[0].innerText.trim();
        if (coordsLookup[name]) {
          map.setView(coordsLookup[name], 11);
          // open popup by finding layer
          markersGroup.eachLayer(layer=>{
            if (layer.getLatLng && layer.getLatLng().lat === coordsLookup[name][0] && layer.getLatLng().lng === coordsLookup[name][1]){
              layer.openPopup();
            }
          });
        } else {
          alert("No map coordinates available for " + name + ". (This map uses approximate centroids.)");
        }
      });
    });

    // basic search
    document.getElementById('searchBox').addEventListener('keydown', function(e){
      if (e.key === "Enter"){
        const q = this.value.trim().toLowerCase();
        if (!q) return;
        // find first matching row
        const rows = Array.from(document.querySelectorAll('#dataTable tbody tr'));
        const found = rows.find(r => r.cells[0].innerText.toLowerCase().includes(q));
        if (found){
          found.scrollIntoView({behavior:'smooth', block:'center'});
          found.classList.add('highlight');
          setTimeout(()=>found.classList.remove('highlight'),1500);
          found.click();
        } else {
          alert("No region/city matched: " + this.value);
        }
      }
    });

    // simple responsive: if map div hidden by sidebar, invalidate size
    setTimeout(()=>map.invalidateSize(),500);

  </script>
</body>
</html>

