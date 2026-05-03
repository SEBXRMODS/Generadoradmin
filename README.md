<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Panel de Keys</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(135deg, #0f172a, #1e293b);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.login, .panel {
  background: #020617;
  padding: 30px;
  border-radius: 12px;
  width: 320px;
  text-align: center;
  box-shadow: 0 0 20px rgba(0,0,0,0.6);
}

input, select {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border-radius: 6px;
  border: none;
}

button {
  width: 100%;
  padding: 10px;
  margin-top: 10px;
  background: #3b82f6;
  border: none;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background: #2563eb;
}

.hidden {
  display: none;
}

.key-box {
  margin-top: 15px;
  background: #111827;
  padding: 10px;
  border-radius: 6px;
  font-weight: bold;
}

.logout {
  background: #ef4444;
}
</style>
</head>

<body>

<div class="login" id="loginBox">
  <h2>Login Admin</h2>
  <input type="text" id="user" placeholder="Usuario">
  <input type="password" id="pass" placeholder="Contraseña">
  <button onclick="login()">Entrar</button>
  <p id="error"></p>
</div>

<div class="panel hidden" id="panelBox">
  <h2>Generador de Keys</h2>

  <select id="duracion">
    <option value="1">1 día</option>
    <option value="2">2 días</option>
    <option value="3">3 días</option>
    <option value="4">4 días</option>
    <option value="5">5 días</option>
    <option value="6">6 días</option>
    <option value="7">7 días</option>
    <option value="14">2 semanas</option>
    <option value="30">1 mes</option>
    <option value="365">1 año</option>
  </select>

  <button onclick="generar()">Generar Key</button>
  <button class="logout" onclick="logout()">Cerrar sesión</button>

  <div id="resultado"></div>
</div>

<script>
const ADMIN_USER = "admin";
const ADMIN_PASS = "1234";

let intervalo = null;

// LOGIN
function login() {
  const u = document.getElementById("user").value;
  const p = document.getElementById("pass").value;

  if (u === ADMIN_USER && p === ADMIN_PASS) {
    document.getElementById("loginBox").classList.add("hidden");
    document.getElementById("panelBox").classList.remove("hidden");
  } else {
    document.getElementById("error").innerText = "Credenciales incorrectas";
  }
}

function logout() {
  document.getElementById("panelBox").classList.add("hidden");
  document.getElementById("loginBox").classList.remove("hidden");
}

// GENERAR KEY
function generarKey() {
  const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  function bloque() {
    let str = "";
    for (let i = 0; i < 4; i++) {
      str += chars[Math.floor(Math.random() * chars.length)];
    }
    return str;
  }

  return `${bloque()}-${bloque()}-${bloque()}`;
}

// FORMATO TIEMPO
function formatearTiempo(ms) {
  let segundos = Math.floor(ms / 1000);

  const dias = Math.floor(segundos / (3600 * 24));
  segundos %= (3600 * 24);

  const horas = Math.floor(segundos / 3600);
  segundos %= 3600;

  const minutos = Math.floor(segundos / 60);
  segundos %= 60;

  return `${dias}d ${horas}h ${minutos}m ${segundos}s`;
}

// GENERAR + CONTADOR
function generar() {
  if (intervalo) clearInterval(intervalo);

  const select = document.getElementById("duracion");
  const dias = parseInt(select.value);
  const texto = select.options[select.selectedIndex].text;

  const key = generarKey();

  const ahora = new Date();
  const expiracion = new Date(ahora.getTime() + dias * 24 * 60 * 60 * 1000);

  const resultado = document.getElementById("resultado");

  resultado.innerHTML = `
    <div class="key-box">
      KEY: ${key} <br>
      DURACIÓN: ${texto} <br>
      EXPIRA EN: <span id="contador"></span>
    </div>
  `;

  intervalo = setInterval(() => {
    const ahora = new Date();
    const restante = expiracion - ahora;

    if (restante <= 0) {
      document.getElementById("contador").innerText = "⛔ Expirada";
      clearInterval(intervalo);
      return;
    }

    document.getElementById("contador").innerText = formatearTiempo(restante);
  }, 1000);
}
</script>

</body>
</html>
