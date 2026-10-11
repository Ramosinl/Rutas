<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Mi Viaje - Argentina</title>
  
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  
  <style>
    :root {
      --primary: #0056b3;
      --primary-dark: #003d80;
      --success: #1b5e20;
      --warning: #e65100;
      --bg: #f8f9fa;
      --text: #1a1a1a;
      --border: #444444;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      background: var(--bg);
      color: var(--text);
      display: flex;
      flex-direction: column;
      height: 100vh;
      overflow: hidden;
    }

    /* Solapas de Navegación Grandes */
    nav {
      display: flex;
      background: #ffffff;
      border-bottom: 2px solid #ccc;
      box-shadow: 0 2px 6px rgba(0,0,0,0.08);
      z-index: 1000;
      min-height: 56px;
    }

    .tab-btn {
      flex: 1;
      padding: 14px 6px;
      font-size: 1.05rem;
      font-weight: 700;
      border: none;
      background: transparent;
      color: #555;
      border-bottom: 4px solid transparent;
      cursor: pointer;
    }

    .tab-btn.active {
      color: var(--primary);
      border-bottom-color: var(--primary);
      background: #eef5fc;
    }

    /* Contenedor de Vistas */
    .view-container {
      flex: 1;
      position: relative;
      overflow: hidden;
    }

    .view {
      display: none;
      height: 100%;
      overflow-y: auto;
      padding: 16px;
    }

    .view.active {
      display: block;
    }

    #viewMap {
      padding: 0;
    }

    /* Formularios y Botones Accesibles */
    .field {
      margin-bottom: 18px;
    }

    label {
      display: block;
      font-size: 1.15rem;
      font-weight: 700;
      margin-bottom: 8px;
      color: #111;
    }

    input {
      width: 100%;
      min-height: 56px;
      padding: 14px;
      font-size: 1.15rem;
      border: 2px solid var(--border);
      border-radius: 12px;
      background: #ffffff;
      color: #000;
      outline: none;
    }

    input:focus {
      border-color: var(--primary);
      background: #f0f7ff;
    }

    .stop-row {
      display: flex;
      gap: 10px;
      margin-bottom: 12px;
    }

    .btn {
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 56px;
      width: 100%;
      font-size: 1.15rem;
      font-weight: 700;
      border-radius: 12px;
      border: none;
      cursor: pointer;
      margin-top: 10px;
      padding: 12px;
    }

    .btn-main {
      background: var(--primary);
      color: #ffffff;
    }

    .btn-secondary {
      background: #e2e8f0;
      color: #1a202c;
      border: 2px solid #cbd5e1;
    }

    .btn-save {
      background: var(--success);
      color: #ffffff;
    }

    .btn-remove {
      width: 56px;
      min-height: 56px;
      background: #fee2e2;
      color: #b91c1c;
      border: 2px solid #ef4444;
      font-size: 1.5rem;
      border-radius: 12px;
    }

    /* Mapa y Resumen Flotante */
    #map {
      width: 100%;
      height: 100%;
    }

    .floating-card {
      position: absolute;
      bottom: 16px;
      left: 12px;
      right: 12px;
      background: #ffffff;
      padding: 16px;
      border-radius: 16px;
      box-shadow: 0 4px 18px rgba(0,0,0,0.25);
      border: 2px solid var(--primary);
      z-index: 999;
      display: none;
    }

    .floating-card h3 {
      font-size: 1.25rem;
      margin-bottom: 8px;
      color: var(--primary);
    }

    .route-info {
      font-size: 1.15rem;
      font-weight: 600;
      line-height: 1.5;
      margin-bottom: 10px;
    }

    .fuel-info {
      background: #fff8e1;
      border: 1px solid var(--warning);
      color: #8c3b00;
      padding: 10px;
      border-radius: 8px;
      font-size: 1.05rem;
      line-height: 1.4;
      margin-bottom: 12px;
    }

    /* Lista de Guardados */
    .trip-card {
      background: #ffffff;
      border: 2px solid #ddd;
      border-radius: 14px;
      padding: 16px;
      margin-bottom: 14px;
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .trip-card strong {
      font-size: 1.2rem;
    }

    .trip-card p {
      font-size: 1.05rem;
      color: #444;
    }
  </style>
</head>
<body>

  <!-- Barra de Solapas -->
  <nav>
    <button class="tab-btn active" onclick="switchTab('plan')">1. Planificar</button>
    <button class="tab-btn" onclick="switchTab('map')">2. Ver Mapa</button>
    <button class="tab-btn" onclick="switchTab('saved')">3. Guardados</button>
  </nav>

  <div class="view-container">

    <!-- VISTA 1: Planificar -->
    <section id="viewPlan" class="view active">
      <div class="field">
        <label for="originInput">Ciudad de Salida (Origen)</label>
        <input type="text" id="originInput" placeholder="Ej: Bahía Blanca" />
      </div>

      <div id="stopsContainer"></div>

      <button type="button" class="btn btn-secondary" id="btnAddStop">
        + Agregar parada intermedia
      </button>

      <div class="field" style="margin-top: 20px;">
        <label for="destInput">Ciudad de Llegada (Destino)</label>
        <input type="text" id="destInput" placeholder="Ej: Bariloche" />
      </div>

      <button type="button" class="btn btn-main" id="btnCalculate" style="margin-top: 24px;">
        Calcular Ruta
      </button>
    </section>

    <!-- VISTA 2: Mapa y Resumen -->
    <section id="viewMap" class="view">
      <div id="map"></div>
      
      <div id="floatingCard" class="floating-card">
        <h3>Datos del Viaje</h3>
        <div class="route-info" id="routeText"></div>
        <div class="fuel-info" id="fuelText"></div>
        <button type="button" class="btn btn-save" id="btnSaveTrip">Guardar este viaje</button>
      </div>
    </section>

    <!-- VISTA 3: Viajes Guardados -->
    <section id="viewSaved" class="view">
      <h2 style="font-size: 1.3rem; margin-bottom: 16px;">Mis Viajes Guardados</h2>
      <div id="savedListContainer"></div>
    </section>

  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <script>
    // Inicializar mapa centrado en Argentina
    const map = L.map('map', { zoomControl: true }).setView([-38.4161, -63.6167], 5);

    L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {
      maxZoom: 18,
      attribution: '&copy; OpenStreetMap'
    }).addTo(map);

    let routeLayer = null;
    let markersLayer = L.layerGroup().addTo(map);
    let activeTripData = null;

    // Control de solapas
    function switchTab(tabName) {
      document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
      document.querySelectorAll('.view').forEach(view => view.classList.remove('active'));

      if (tabName === 'plan') {
        document.querySelector("button[onclick=\"switchTab('plan')\"]").classList.add('active');
        document.getElementById('viewPlan').classList.add('active');
      } else if (tabName === 'map') {
        document.querySelector("button[onclick=\"switchTab('map')\"]").classList.add('active');
        document.getElementById('viewMap').classList.add('active');
        setTimeout(() => map.invalidateSize(), 200); // Redibuja el mapa al cambiar de vista
      } else if (tabName === 'saved') {
        document.querySelector("button[onclick=\"switchTab('saved')\"]").classList.add('active');
        document.getElementById('viewSaved').classList.add('active');
        renderSavedTrips();
      }
    }

    // Agregar paradas intermedias
    document.getElementById('btnAddStop').addEventListener('click', () => {
      const div = document.createElement('div');
      div.className = 'stop-row';
      div.innerHTML = `
        <input type="text" class="intermediate-input" placeholder="Parada (Ej: Choele Choel)" />
        <button type="button" class="btn-remove" title="Quitar">✕</button>
      `;
      div.querySelector('.btn-remove').addEventListener('click', () => div.remove());
      document.getElementById('stopsContainer').appendChild(div);
    });

    // Geocodificación
    async function searchLocation(query) {
      if (!query.trim()) return null;
      const url = `https://nominatim.openstreetmap.org/search?format=json&countrycodes=ar&q=${encodeURIComponent(query)}`;
      try {
        const res = await fetch(url);
        const data = await res.json();
        if (data && data.length > 0) {
          return {
            name: data[0].display_name.split(',')[0],
            lat: parseFloat(data[0].lat),
            lon: parseFloat(data[0].lon)
          };
        }
      } catch (e) {
        console.error(e);
      }
      return null;
    }

    // Cálculo de ruta
    document.getElementById('btnCalculate').addEventListener('click', async () => {
      const originText = document.getElementById('originInput').value;
      const destText = document.getElementById('destInput').value;
      const stopInputs = Array.from(document.querySelectorAll('.intermediate-input'));

      if (!originText || !destText) {
        alert("Por favor indicá al menos la ciudad de salida y la de llegada.");
        return;
      }

      const btn = document.getElementById('btnCalculate');
      btn.innerText = "Calculando trayecto...";
      btn.disabled = true;

      try {
        const originCoord = await searchLocation(originText);
        const destCoord = await searchLocation(destText);

        if (!originCoord || !destCoord) {
          alert("No se encontró alguna de las ciudades. Verificá cómo está escrita.");
          return;
        }

        const points = [originCoord];
        for (const input of stopInputs) {
          if (input.value.trim()) {
            const coord = await searchLocation(input.value);
            if (coord) points.push(coord);
          }
        }
        points.push(destCoord);

        await processRoute(points);
        switchTab('map'); // Abre automáticamente el mapa al terminar el cálculo
      } catch (err) {
        alert("Hubo un problema al calcular la ruta. Comprobá la conexión e intentá de nuevo.");
      } finally {
        btn.innerText = "Calcular Ruta";
        btn.disabled = false;
      }
    });

    async function processRoute(points) {
      markersLayer.clearLayers();
      if (routeLayer) map.removeLayer(routeLayer);

      const coordStr = points.map(p => `${p.lon},${p.lat}`).join(';');
      const url = `https://router.project-osrm.org/route/v1/driving/${coordStr}?overview=full&geometries=geojson`;

      const res = await fetch(url);
      const data = await res.json();

      if (!data.routes || data.routes.length === 0) {
        alert("No se encontró camino terrestre directo entre esas localidades.");
        return;
      }

      const route = data.routes[0];
      const distanceKm = Math.round(route.distance / 1000);
      const totalMinutes = Math.round((distanceKm / 85) * 60);
      const hours = Math.floor(totalMinutes / 60);
      const mins = totalMinutes % 60;

      // Dibujar línea gruesa y contrastada
      routeLayer = L.geoJSON({ type: "Feature", geometry: route.geometry }, {
        style: { color: '#0056b3', weight: 8, opacity: 0.9 }
      }).addTo(map);

      map.fitBounds(routeLayer.getBounds(), { padding: [50, 50] });

      // Marcadores grandes
      points.forEach((p, idx) => {
        let label = `Parada: ${p.name}`;
        let color = "#e65100";
        if (idx === 0) { label = `Salida: ${p.name}`; color = "#1b5e20"; }
        if (idx === points.length - 1) { label = `Llegada: ${p.name}`; color = "#b71c1c"; }

        L.circleMarker([p.lat, p.lon], {
          radius: 12,
          fillColor: color,
          color: "#ffffff",
          weight: 3,
          fillOpacity: 1
        }).addTo(markersLayer).bindPopup(`<b>${label}</b>`);
      });

      // Cálculo preventivo de combustible
      const stopsCount = Math.floor(distanceKm / 280);
      let fuelNotice = stopsCount > 0
        ? `⛽ <strong>Carga de combustible:</strong> Conviene hacer <strong>${stopsCount} parada(s)</strong> técnica(s) cada 250–300 km.`
        : `⛽ <strong>Combustible:</strong> Viaje directo con un tanque completo.`;

      document.getElementById('routeText').innerHTML = `
        Salida: <strong>${points[0].name}</strong><br>
        Llegada: <strong>${points[points.length - 1].name}</strong><br>
        Distancia: <strong>${distanceKm} km</strong> | Tiempo: <strong>${hours}h ${mins}min</strong> aprox.
      `;
      document.getElementById('fuelText').innerHTML = fuelNotice;
      document.getElementById('floatingCard').style.display = 'block';

      activeTripData = {
        origin: points[0].name,
        dest: points[points.length - 1].name,
        distance: distanceKm,
        duration: `${hours}h ${mins}min`
      };
    }

    // Almacenamiento local
    document.getElementById('btnSaveTrip').addEventListener('click', () => {
      if (!activeTripData) return;
      const list = JSON.parse(localStorage.getItem('mis_rutas') || "[]");
      list.push(activeTripData);
      localStorage.setItem('mis_rutas', JSON.stringify(list));
      alert("¡Viaje guardado!");
      switchTab('saved');
    });

    function renderSavedTrips() {
      const container = document.getElementById('savedListContainer');
      container.innerHTML = "";
      const list = JSON.parse(localStorage.getItem('mis_rutas') || "[]");

      if (list.length === 0) {
        container.innerHTML = "<p style='font-size:1.1rem; color:#666;'>No hay viajes guardados todavía.</p>";
        return;
      }

      list.forEach((trip, index) => {
        const card = document.createElement('div');
        card.className = 'trip-card';
        card.innerHTML = `
          <strong>${trip.origin} ➔ ${trip.dest}</strong>
          <p>Distancia: ${trip.distance} km | Tiempo: ${trip.duration}</p>
          <button class="btn btn-secondary" style="min-height:44px; font-size:1rem; margin-top:4px;" onclick="deleteTrip(${index})">Borrar de la lista</button>
        `;
        container.appendChild(card);
      });
    }

    window.deleteTrip = function(index) {
      const list = JSON.parse(localStorage.getItem('mis_rutas') || "[]");
      list.splice(index, 1);
      localStorage.setItem('mis_rutas', JSON.stringify(list));
      renderSavedTrips();
    };
  </script>
</body>
</html>
