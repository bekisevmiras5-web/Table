"""
PathWise — Logistics Route Optimizer
=====================================
Single-file Flask app. Run:
    pip install -r requirements.txt
    python app.py

Deploy to Render / Railway:
    Start command: gunicorn app:app
"""

import os
from datetime import datetime
from math import atan2, cos, radians, sin, sqrt

import networkx as nx
import requests as req
from flask import Flask, jsonify, render_template_string, request
from flask_sqlalchemy import SQLAlchemy

# ══════════════════════════════════════════
#  APP & DATABASE SETUP
# ══════════════════════════════════════════

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = (
    f"sqlite:///{os.path.join(BASE_DIR, 'pathwise.db')}"
)
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)


class DeliveryTask(db.Model):
    __tablename__ = "delivery_tasks"

    id         = db.Column(db.Integer, primary_key=True)
    address    = db.Column(db.String(500), nullable=False)
    lat        = db.Column(db.Float, nullable=False)
    lng        = db.Column(db.Float, nullable=False)
    priority   = db.Column(db.String(20), default="medium")
    status     = db.Column(db.String(20), default="pending")
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def to_dict(self):
        return {
            "id":         self.id,
            "address":    self.address,
            "lat":        self.lat,
            "lng":        self.lng,
            "priority":   self.priority,
            "status":     self.status,
            "created_at": self.created_at.isoformat(),
        }


# ══════════════════════════════════════════
#  SAMPLE DATA
# ══════════════════════════════════════════

SAMPLE_POINTS = [
    {"address": "London Bridge, London, UK",        "lat": 51.5079, "lng": -0.0877, "priority": "high"},
    {"address": "Tower of London, London, UK",       "lat": 51.5081, "lng": -0.0759, "priority": "medium"},
    {"address": "St. Paul's Cathedral, London, UK", "lat": 51.5138, "lng": -0.0984, "priority": "high"},
    {"address": "Covent Garden, London, UK",         "lat": 51.5129, "lng": -0.1243, "priority": "low"},
    {"address": "Trafalgar Square, London, UK",      "lat": 51.5080, "lng": -0.1281, "priority": "medium"},
    {"address": "Westminster Bridge, London, UK",    "lat": 51.5007, "lng": -0.1246, "priority": "high"},
]


# ══════════════════════════════════════════
#  TSP OPTIMIZATION LOGIC  (NetworkX)
# ══════════════════════════════════════════

def haversine(lat1, lng1, lat2, lng2):
    """Real-world distance in km between two lat/lng points."""
    R = 6371.0
    lat1, lng1, lat2, lng2 = map(radians, [lat1, lng1, lat2, lng2])
    dlat = lat2 - lat1
    dlng = lng2 - lng1
    a = sin(dlat / 2) ** 2 + cos(lat1) * cos(lat2) * sin(dlng / 2) ** 2
    return R * 2 * atan2(sqrt(a), sqrt(1 - a))


def optimize_route(points):
    """
    Nearest-Neighbour Greedy Heuristic for TSP.
    Builds a complete weighted graph with NetworkX,
    then greedily visits the closest unvisited node.
    """
    n = len(points)
    if n < 2:
        return {"error": "Need at least 2 points"}

    G = nx.Graph()
    for i in range(n):
        G.add_node(i, **points[i])

    for i in range(n):
        for j in range(i + 1, n):
            dist = haversine(
                points[i]["lat"], points[i]["lng"],
                points[j]["lat"], points[j]["lng"],
            )
            G.add_edge(i, j, weight=dist)

    # Greedy nearest-neighbour starting from node 0
    visited = [False] * n
    path    = [0]
    visited[0] = True

    for _ in range(n - 1):
        current   = path[-1]
        best_next = min(
            (j for j in range(n) if not visited[j]),
            key=lambda j: G[current][j]["weight"],
        )
        path.append(best_next)
        visited[best_next] = True

    # Total distance (closed loop)
    total = sum(G[path[i]][path[i + 1]]["weight"] for i in range(n - 1))
    total += G[path[-1]][path[0]]["weight"]

    optimized = [points[i] for i in path]

    steps = []
    for i in range(n):
        nxt      = (i + 1) % n
        seg_dist = G[path[i]][path[nxt]]["weight"]
        steps.append({
            "step":               i + 1,
            "stop_id":            optimized[i]["id"],
            "address":            optimized[i]["address"],
            "lat":                optimized[i]["lat"],
            "lng":                optimized[i]["lng"],
            "distance_to_next_km": round(seg_dist, 3),
            "next_address":       optimized[nxt]["address"] if i < n - 1 else optimized[0]["address"],
        })

    return {
        "optimized_route":      optimized,
        "total_distance_km":    round(total, 3),
        "estimated_time_minutes": round((total / 30.0) * 60),
        "num_stops":            n,
        "steps":                steps,
        "algorithm":            "Nearest Neighbour Greedy Heuristic (TSP Approximation)",
        "graph_edges":          len(G.edges()),
        "graph_nodes":          len(G.nodes()),
    }


# ══════════════════════════════════════════
#  GEOCODING  (OpenStreetMap Nominatim)
# ══════════════════════════════════════════

def geocode_address(address):
    try:
        r = req.get(
            "https://nominatim.openstreetmap.org/search",
            params={"q": address, "format": "json", "limit": 1},
            headers={"User-Agent": "PathWise-Logistics/1.0"},
            timeout=6,
        )
        data = r.json()
        if data:
            return float(data[0]["lat"]), float(data[0]["lon"])
    except Exception:
        pass
    return None, None


# ══════════════════════════════════════════
#  HTML TEMPLATE  (embedded)
# ══════════════════════════════════════════

HTML = r"""<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
  <title>PathWise — Оптимизатор маршрутов</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link  rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            lime:   { DEFAULT: '#32CD32', dim: '#1a6b1a', bright: '#45e645' },
            purple: { DEFAULT: '#BF00FF', dim: '#5c0079', bright: '#d400ff' },
            dark:   { 900:'#050505', 800:'#0d0d0d', 700:'#141414', 600:'#1c1c1c', 500:'#222222', 400:'#2a2a2a' }
          },
          fontFamily: { mono: ['Courier New','Lucida Console','monospace'] }
        }
      }
    }
  </script>
  <style>
    * { box-sizing: border-box; }
    body { font-family: 'Courier New','Lucida Console',monospace; }

    ::-webkit-scrollbar { width:3px; height:3px; }
    ::-webkit-scrollbar-track { background:#0d0d0d; }
    ::-webkit-scrollbar-thumb { background:#1a6b1a; border-radius:2px; }

    #map { width:100%; height:100%; background:#080808; }
    .leaflet-container { background:#080808 !important; }
    .leaflet-tile { filter:invert(1) hue-rotate(200deg) brightness(0.82) saturate(0.55); }
    .leaflet-control-zoom { border:1px solid #222 !important; }
    .leaflet-control-zoom a { background:#0d0d0d !important; color:#32CD32 !important; border-color:#222 !important; }
    .leaflet-control-attribution { background:rgba(5,5,5,.8) !important; color:#333 !important; font-size:9px !important; }
    .leaflet-control-attribution a { color:#444 !important; }
    .leaflet-popup-content-wrapper { background:#0d0d0d !important; border:1px solid #222 !important; border-radius:4px !important; color:#e0e0e0 !important; }
    .leaflet-popup-tip { background:#0d0d0d !important; }

    .spinner { display:inline-block; width:12px; height:12px; border:2px solid transparent; border-top-color:currentColor; border-radius:50%; animation:spin .6s linear infinite; }
    @keyframes spin { to { transform:rotate(360deg); } }

    .tab-btn.active { background:#32CD32 !important; color:#000 !important; border-color:#32CD32 !important; }

    .tag { font-size:9px; letter-spacing:1px; padding:2px 6px; border-radius:2px; text-transform:uppercase; font-weight:700; }
    .tag-high     { background:rgba(255,80,80,.15);  color:#ff5050; border:1px solid rgba(255,80,80,.25); }
    .tag-medium   { background:rgba(255,165,0,.12);  color:#ffa500; border:1px solid rgba(255,165,0,.2); }
    .tag-low      { background:rgba(100,100,255,.12);color:#8888ff; border:1px solid rgba(100,100,255,.2); }
    .tag-pending  { background:rgba(100,100,100,.15);color:#888;    border:1px solid #333; }
    .tag-in_progress { background:rgba(50,205,50,.1);color:#32CD32; border:1px solid rgba(50,205,50,.2); }
    .tag-delivered   { background:rgba(76,175,80,.08);color:#4caf50;border:1px solid rgba(76,175,80,.15); }

    input:focus, select:focus { outline:none; border-color:#32CD32 !important; box-shadow:0 0 0 1px #1a6b1a; }

    .btn-lime:hover   { background:#45e645; box-shadow:0 0 14px rgba(50,205,50,.35); }
    .btn-lime:disabled{ opacity:.4; cursor:not-allowed; }
    .btn-purple:hover { background:#d400ff; box-shadow:0 0 14px rgba(191,0,255,.35); }
    .btn-ghost:hover  { border-color:#555; color:#ccc; }

    #toast {
      position:fixed; bottom:20px; left:50%; transform:translateX(-50%);
      background:#0d0d0d; border:1px solid #222; color:#e0e0e0;
      font-family:'Courier New',monospace; font-size:11px; letter-spacing:1px;
      padding:10px 20px; border-radius:4px; z-index:9999;
      opacity:0; transition:opacity .2s; pointer-events:none;
      white-space:nowrap; max-width:90vw;
    }
    #toast.show    { opacity:1; }
    #toast.success { border-color:#32CD32; color:#32CD32; }
    #toast.error   { border-color:#cc2222; color:#ff5555; }

    .glow-line { height:1px; background:linear-gradient(90deg,transparent,#32CD3222,#32CD3244,#32CD3222,transparent); }

    @media (max-width:767px)                    { #map-panel { height:45vh; } }
    @media (min-width:768px) and (max-width:1023px) { #map-panel { height:50vh; } }
    @media (min-width:1024px) {
      #desktop-layout { display:grid; grid-template-columns:320px 1fr 300px; height:calc(100vh - 56px); }
      #mobile-tabs    { display:none; }
      #map-panel      { height:100%; }
      .desktop-panel  { display:flex !important; flex-direction:column; overflow:hidden; }
      .panel-scroll   { flex:1; overflow-y:auto; }
    }
  </style>
</head>
<body class="bg-dark-900 text-gray-200 font-mono min-h-screen flex flex-col">

<!-- NAVBAR -->
<nav class="bg-dark-800 border-b border-dark-500 px-3 md:px-5 h-14 flex items-center justify-between flex-shrink-0 z-50">
  <div class="flex items-center gap-2 md:gap-3">
    <div class="w-7 h-7 rounded flex items-center justify-center text-sm font-black flex-shrink-0"
         style="background:linear-gradient(135deg,#32CD32,#BF00FF)">⬡</div>
    <div class="flex items-baseline gap-1">
      <span class="text-lime   font-black text-base md:text-lg tracking-widest">PATH</span>
      <span class="text-purple font-black text-base md:text-lg tracking-widest">WISE</span>
      <span class="hidden sm:inline text-gray-600 text-xs tracking-widest ml-1">ЛОГИСТИКА</span>
    </div>
  </div>
  <div class="flex items-center gap-3 md:gap-5 text-xs tracking-widest">
    <div class="hidden sm:flex gap-4">
      <span class="text-gray-600">СТОП: <strong class="text-lime"          id="nav-total">0</strong></span>
      <span class="text-gray-600">ОЖИД: <strong class="text-yellow-400"    id="nav-pending">0</strong></span>
      <span class="hidden md:inline text-gray-600">ДОСТВ: <strong class="text-green-400" id="nav-delivered">0</strong></span>
    </div>
    <div class="flex items-center gap-1">
      <span class="inline-block w-2 h-2 rounded-full bg-lime animate-pulse"></span>
      <span class="text-lime text-xs tracking-wider" id="nav-status">ГОТОВ</span>
    </div>
  </div>
</nav>
<div class="glow-line"></div>

<!-- MOBILE TABS -->
<div id="mobile-tabs" class="flex lg:hidden bg-dark-800 border-b border-dark-500 flex-shrink-0">
  <button onclick="switchTab('stops')" class="tab-btn active flex-1 py-2.5 text-xs tracking-widest uppercase border-r border-dark-500 bg-dark-800 text-gray-400 transition-all" data-tab="stops">📦 Стопы</button>
  <button onclick="switchTab('map')"   class="tab-btn        flex-1 py-2.5 text-xs tracking-widest uppercase border-r border-dark-500 bg-dark-800 text-gray-400 transition-all" data-tab="map">🗺 Карта</button>
  <button onclick="switchTab('route')" class="tab-btn        flex-1 py-2.5 text-xs tracking-widest uppercase                           bg-dark-800 text-gray-400 transition-all" data-tab="route">📊 Маршрут</button>
</div>

<!-- MAIN LAYOUT -->
<div id="desktop-layout" class="flex-1 overflow-hidden flex flex-col lg:grid">

  <!-- ── ЛЕВАЯ ПАНЕЛЬ ── -->
  <div id="panel-stops" class="desktop-panel bg-dark-800 border-r border-dark-500 flex flex-col overflow-hidden">

    <div class="px-4 py-2.5 border-b border-dark-500 flex items-center justify-between flex-shrink-0">
      <span class="text-xs tracking-widest text-gray-500 uppercase">Точки доставки</span>
      <span class="bg-dark-600 border border-dark-500 text-lime text-xs px-2 py-0.5 rounded" id="task-count">0</span>
    </div>

    <div class="p-3 border-b border-dark-500 flex-shrink-0">
      <!-- Адрес -->
      <div class="mb-2">
        <label class="block text-xs tracking-widest text-gray-600 mb-1 uppercase">Адрес доставки</label>
        <input type="text" id="inp-address" placeholder="напр. ул. Пушкина, Москва"
               class="w-full bg-dark-700 border border-dark-500 text-gray-200 font-mono text-xs px-3 py-2 rounded transition-all" />
      </div>

      <!-- Скрытые координаты -->
      <div class="mb-2">
        <button type="button" onclick="toggleAdvanced()"
                class="flex items-center gap-1.5 text-xs text-gray-600 hover:text-gray-400 transition-all tracking-wider uppercase">
          <span id="adv-arrow">▶</span>
          <span>Координаты вручную</span>
          <span class="text-gray-700 text-xs normal-case tracking-normal">(необязательно)</span>
        </button>
        <div id="adv-panel" class="hidden mt-2 grid grid-cols-2 gap-2">
          <div>
            <label class="block text-xs tracking-widest text-gray-600 mb-1 uppercase">Широта</label>
            <input type="number" id="inp-lat" placeholder="55.7558" step="0.0001"
                   class="w-full bg-dark-700 border border-dark-500 text-gray-200 font-mono text-xs px-3 py-2 rounded transition-all" />
          </div>
          <div>
            <label class="block text-xs tracking-widest text-gray-600 mb-1 uppercase">Долгота</label>
            <input type="number" id="inp-lng" placeholder="37.6173" step="0.0001"
                   class="w-full bg-dark-700 border border-dark-500 text-gray-200 font-mono text-xs px-3 py-2 rounded transition-all" />
          </div>
          <p class="col-span-2 text-xs text-gray-700">Если геокодер не нашёл адрес — введите координаты вручную.</p>
        </div>
      </div>

      <!-- Приоритет -->
      <div class="mb-3">
        <label class="block text-xs tracking-widest text-gray-600 mb-1 uppercase">Приоритет</label>
        <select id="inp-priority"
                class="w-full bg-dark-700 border border-dark-500 text-gray-200 font-mono text-xs px-3 py-2 rounded transition-all">
          <option value="high">Высокий</option>
          <option value="medium" selected>Средний</option>
          <option value="low">Низкий</option>
        </select>
      </div>

      <button onclick="addDelivery()" id="btn-add"
              class="btn-lime w-full bg-lime text-black font-black text-xs tracking-widest uppercase py-2.5 rounded transition-all">
        + Добавить точку
      </button>

      <div class="grid grid-cols-2 gap-2 mt-2">
        <button onclick="seedData()"
                class="btn-ghost bg-transparent border border-dark-500 text-gray-500 text-xs tracking-wider uppercase py-2 rounded transition-all">
          ⟳ Демо-данные
        </button>
        <button onclick="clearAll()"
                class="btn-ghost bg-transparent border border-dark-500 text-gray-500 text-xs tracking-wider uppercase py-2 rounded transition-all hover:border-red-800 hover:text-red-500">
          ✕ Очистить
        </button>
      </div>
    </div>

    <!-- Список стопов -->
    <div class="panel-scroll flex-1 overflow-y-auto p-3" id="delivery-list">
      <div class="flex flex-col items-center justify-center py-10 gap-2 text-gray-700">
        <div class="text-4xl opacity-30">📦</div>
        <div class="text-xs tracking-widest">Нет точек доставки</div>
        <div class="text-xs">Добавьте адрес или загрузите демо</div>
      </div>
    </div>

    <div class="p-3 border-t border-dark-500 flex-shrink-0">
      <button onclick="optimizeRoute()" id="btn-optimize"
              class="btn-purple w-full text-white font-black text-xs tracking-widest uppercase py-3 rounded transition-all"
              style="background:#BF00FF">
        ▶ Рассчитать оптимальный маршрут
      </button>
    </div>
  </div>

  <!-- ── КАРТА ── -->
  <div id="panel-map" class="relative flex flex-col overflow-hidden bg-dark-900">
    <div id="map-panel" class="flex-1 relative">
      <div id="map" style="position:absolute;inset:0;"></div>

      <div class="absolute top-2 left-2 z-[1000] flex flex-col gap-1.5">
        <div class="bg-dark-900 bg-opacity-90 border border-dark-500 rounded px-2.5 py-1.5 text-xs text-gray-600 backdrop-blur-sm">
          OpenStreetMap · Leaflet.js
        </div>
        <div class="bg-dark-900 bg-opacity-90 border border-dark-500 rounded px-2.5 py-1.5 text-xs backdrop-blur-sm" id="map-status">
          <strong class="text-lime">0</strong> <span class="text-gray-600">маркеров</span>
        </div>
      </div>

      <div class="absolute bottom-3 left-1/2 -translate-x-1/2 z-[1000] lg:hidden">
        <button onclick="optimizeRoute()" id="btn-optimize-map"
                class="btn-purple text-white font-black text-xs tracking-wider uppercase px-5 py-2.5 rounded shadow-lg transition-all"
                style="background:#BF00FF;box-shadow:0 0 20px rgba(191,0,255,.4)">
          ▶ Рассчитать маршрут
        </button>
      </div>
    </div>
  </div>

  <!-- ── ПРАВАЯ ПАНЕЛЬ ── -->
  <div id="panel-route" class="desktop-panel bg-dark-800 border-l border-dark-500 flex flex-col overflow-hidden">

    <div class="px-4 py-2.5 border-b border-dark-500 flex items-center justify-between flex-shrink-0">
      <span class="text-xs tracking-widest text-gray-500 uppercase">Анализ маршрута</span>
      <span class="text-xs tracking-widest px-2 py-0.5 rounded border border-dark-500 text-gray-600" id="route-badge">ОЖИДАНИЕ</span>
    </div>

    <div class="p-3 border-b border-dark-500 flex-shrink-0">
      <div class="grid grid-cols-2 gap-2 mb-2">
        <div class="bg-dark-700 border border-dark-500 rounded p-2.5 text-center">
          <div class="text-xl font-black text-lime   leading-none mb-1" id="res-distance">—</div>
          <div class="text-xs text-gray-600 tracking-widest uppercase">км всего</div>
        </div>
        <div class="bg-dark-700 border border-dark-500 rounded p-2.5 text-center">
          <div class="text-xl font-black text-purple leading-none mb-1" id="res-time">—</div>
          <div class="text-xs text-gray-600 tracking-widest uppercase">мин (оценка)</div>
        </div>
        <div class="bg-dark-700 border border-dark-500 rounded p-2.5 text-center">
          <div class="text-xl font-black text-lime   leading-none mb-1" id="res-stops">—</div>
          <div class="text-xs text-gray-600 tracking-widest uppercase">остановок</div>
        </div>
        <div class="bg-dark-700 border border-dark-500 rounded p-2.5 text-center">
          <div class="text-xl font-black text-purp