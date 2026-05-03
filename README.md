<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Panel SaaS Admin</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b1220;
  color: white;
}

/* LOGIN */
#login {
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
#dash {
  display: none;
  padding: 20px;
}

.card {
  background: #111a2e;
  padding: 10px;
  border-radius: 10px;
  margin: 5px;
}

.panel {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 20px;
  margin-top: 20px;
}

.section {
  background: #111a2e;
  padding: 15px;
  border-radius: 10px;
}

.item {
  background: #0f172a;
  padding: 10px;
  margin-top: 8px;
  border-radius: 6px;
  display: flex;
  justify-content: space-between;
}

.small {
  font-size: 12px;
  opacity: 0.7;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">
  <div class="box">
    <h2>Admin Login</h2>

    <input id="user" placeholder="Usuario">
    <input id="pass" type="password" placeholder="Contraseña">

    <button onclick="login()">Entrar</button>

    <p id="error"></p>
  </div>
</div>

<!-- DASHBOARD -->
<div id="dash">

  <h2>Panel SaaS</h2>
  <button onclick="logout()">Cerrar sesión</button>

  <div class="panel">

    <!-- CREAR -->
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
      <div id="list"></div>
    </div>

  </div>

</div>

<script>
/* ---------------- LOGIN ---------------- */
const ADMIN_USER = "admin";
const ADMIN_PASS = "1234";

let keys = [];

function login() {
  const u = document.getElementById("user").value;
  const p = document.getElementById("pass").value;

  if (u === ADMIN_USER && p === ADMIN_PASS) {
    document.getElementById("login").style.display = "none";
    document.getElementById("dash").style.display = "block";
  } else {
    document.getElementById("error").innerText = "Credenciales incorrectas";
  }
}

function logout() {
  location.reload();
}

/* ---------------- GENERAR KEY ---------------- */
function genKey() {
  const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  let key = "";

  for (let i = 0; i < 16; i++) {
    key += chars[Math.floor(Math.random() * chars.length)];
    if (i % 4 === 3 && i < 15) key += "-";
  }

  return key;
}

/* ---------------- CREAR KEY ---------------- */
function createKey() {
  const days = parseInt(document.getElementById("duration").value);

  const key = {
    id: Date.now(),
    key: genKey(),
    days: days
  };

  keys.push(key);

  document.getElementById("newKey").innerHTML =
    "KEY: <b>" + key.key + "</b>";

  render();
}

/* ---------------- RENDER ---------------- */
function render() {
  const list = document.getElementById("list");
  list.innerHTML = "";

  keys.forEach(k => {
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
