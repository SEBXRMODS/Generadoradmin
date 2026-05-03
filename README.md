<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>SaaS Admin Panel</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b1220;
  color: white;
}

/* LOGIN */
.login {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.box {
  background: #111a2e;
  padding: 30px;
  border-radius: 12px;
  width: 320px;
  text-align: center;
}

input, select {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
  border-radius: 6px;
  border: none;
}

button {
  width: 100%;
  padding: 10px;
  margin-top: 8px;
  background: #3b82f6;
  border: none;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background: #2563eb;
}

/* DASHBOARD */
.dashboard {
  display: none;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cards {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.card {
  flex: 1;
  background: #111a2e;
  padding: 15px;
  border-radius: 10px;
  text-align: center;
}

/* PANEL */
.panel {
  margin-top: 20px;
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 20px;
}

.section {
  background: #111a2e;
  padding: 15px;
  border-radius: 10px;
}

/* LISTA */
.list {
  max-height: 300px;
  overflow-y: auto;
}

.item {
  background: #0f172a;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 6px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.small {
  font-size: 12px;
  opacity: 0.7;
}

.actions button {
  width: auto;
  padding: 5px 8px;
  margin-left: 5px;
  font-size: 12px;
}

.delete {
  background: #ef4444;
}
.copy {
  background: #10b981;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div class="login" id="login">
  <div class="box">
    <h2>Admin Login</h2>
    <input id="user" placeholder="Usuario">
    <input id="pass" type="password" placeholder="Contraseña">
    <button onclick="login()">Entrar</button>
    <p id="error"></p>
  </div>
</div>

<!-- DASHBOARD -->
<div class="dashboard" id="dash">

  <div class="header">
    <h2>Panel SaaS</h2>
    <button onclick="logout()">Salir</button>
  </div>

  <div class="cards">
    <div class="card">
      <h3 id="totalKeys">0</h3>
      <p>Total Keys</p>
    </div>
    <div class="card">
      <h3 id="activeKeys">0</h3>
      <p>Activas</p>
    </div>
    <div class="card">
      <h3 id="usedKeys">0</h3>
      <p>Usadas</p>
    </div>
  </div>

  <div class="panel">

    <!-- CREAR KEY -->
    <div class="section">
      <h3>Crear Key</h3>

      <select id="duration">
        <option value="1">1 día</option>
        <option value="7">7 días</option>
        <option value="14">2 semanas</option>
        <option value="30">1 mes</option>
        <option value="365">1 año</option>
      </select>

      <button onclick="createKey()">Generar</button>

      <p id="newKey"></p>
    </div>

    <!-- LISTA -->
    <div class="section">
      <h3>Keys</h3>
      <input placeholder="Buscar..." oninput="search(this.value)">

      <div class="list" id="list"></div>
    </div>

  </div>
</div>

<script>
/* ---------------- LOGIN ---------------- */
const ADMIN_USER = "admin";
const ADMIN_PASS = "1234";

function login() {
  const u = user.value;
  const p = pass.value;

  if (u === ADMIN_USER && p === ADMIN_PASS) {
    login.style.display = "none";
    dash.style.display = "block";
    render();
  } else {
    error.innerText = "Credenciales incorrectas";
  }
}

function logout() {
  location.reload();
}

/* ---------------- DB LOCAL (SIMULADO) ---------------- */
let keys = [];

/* ---------------- GENERAR KEY ---------------- */
function genKey() {
  const c = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  let k = "";

  for (let i = 0; i < 16; i++) {
    k += c[Math.floor(Math.random() * c.length)];
    if (i % 4 === 3 && i < 15) k += "-";
  }
  return k;
}

/* ---------------- CREAR KEY ---------------- */
function createKey() {
  const days = parseInt(duration.value);

  const key = {
    id: Date.now(),
    key: genKey(),
    days,
    used: false,
    created: new Date()
  };

  keys.push(key);

  newKey.innerHTML = "KEY: <b>" + key.key + "</b>";

  render();
}

/* ---------------- RENDER ---------------- */
function render() {

  totalKeys.innerText = keys.length;
  activeKeys.innerText = keys.filter(k => !k.used).length;
  usedKeys.innerText = keys.filter(k => k.used).length;

  list.innerHTML = "";

  keys.forEach(k => {

    const div = document.createElement("div");
    div.className = "item";

    div.innerHTML = `
      <div>
        <b>${k.key}</b>
        <div class="small">${k.days} días</div>
      </div>

      <div class="actions">
        <button class="copy" onclick="copyKey('${k.key}')">Copiar</button>
        <button class="delete" onclick="deleteKey(${k.id})">X</button>
      </div>
    `;

    list.appendChild(div);
  });
}

/* ---------------- COPY ---------------- */
function copyKey(k) {
  navigator.clipboard.writeText(k);
  alert("Copiada");
}

/* ---------------- DELETE ---------------- */
function deleteKey(id) {
  keys = keys.filter(k => k.id !== id);
  render();
}

/* ---------------- SEARCH ---------------- */
function search(v) {
  const filtered = keys.filter(k => k.key.includes(v.toUpperCase()));

  list.innerHTML = "";

  filtered.forEach(k => {
    const div = document.createElement("div");
    div.className = "item";

    div.innerHTML = `
      <div>
        <b>${k.key}</b>
        <div class="small">${k.days} días</div>
      </div>
    `;

    list.appendChild(div);
  });
}
</script>

</body>
</html>
