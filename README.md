<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Panel SaaS Admin</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b1220;
  color: white;
}

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
  box-sizing: border-box;
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

#dash {
  display: none;
  padding: 20px;
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
}

.small {
  font-size: 12px;
  opacity: 0.7;
  margin-top: 5px;
}

.status {
  margin-top: 5px;
  font-weight: bold;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">
  <div class="box">

    <h2>Admin Login</h2>

    <input id="user" placeholder="Email">

    <input id="pass" type="password" placeholder="Contraseña">

    <button onclick="login()">
      Entrar
    </button>

    <p id="error"></p>

  </div>
</div>

<!-- DASHBOARD -->
<div id="dash">

  <h2>Panel SaaS</h2>

  <button onclick="logout()">
    Cerrar sesión
  </button>

  <div class="panel">

    <!-- CREAR KEY -->
    <div class="section">

      <h3>Crear Key</h3>

      <select id="duration">

        <option value="1">
          1 día
        </option>

        <option value="7">
          7 días
        </option>

        <option value="14">
          2 semanas
        </option>

        <option value="30">
          1 mes
        </option>

        <option value="365">
          1 año
        </option>

      </select>

      <button onclick="createKey()">
        Generar
      </button>

      <p id="newKey"></p>

    </div>

    <!-- LISTA -->
    <div class="section">

      <h3>Keys Generadas</h3>

      <div id="list"></div>

    </div>

  </div>

</div>

<script type="module">

/* FIREBASE IMPORTS */
import { initializeApp }
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getFirestore,
  collection,
  addDoc,
  getDocs
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

import {
  getAuth,
  signInWithEmailAndPassword,
  onAuthStateChanged,
  signOut
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

/* FIREBASE CONFIG */
const firebaseConfig = {

  apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain: "panelsebxrmods.firebaseapp.com",

  projectId: "panelsebxrmods",

  storageBucket: "panelsebxrmods.firebasestorage.app",

  messagingSenderId: "717339227525",

  appId: "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

/* INICIAR FIREBASE */
const app = initializeApp(firebaseConfig);

const db = getFirestore(app);

const auth = getAuth(app);

/* LOGIN FIREBASE */
window.login = async function () {

  const email =
    document.getElementById("user").value;

  const password =
    document.getElementById("pass").value;

  try {

    await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

  } catch (err) {

    document.getElementById("error").innerText =
      err.message;

  }

};

/* SESION */
onAuthStateChanged(auth, user => {

  if (user) {

    document.getElementById("login").style.display =
      "none";

    document.getElementById("dash").style.display =
      "block";

    loadKeys();

  } else {

    document.getElementById("login").style.display =
      "flex";

    document.getElementById("dash").style.display =
      "none";

  }

});

/* LOGOUT */
window.logout = async function () {

  await signOut(auth);

};

/* GENERADOR */
function genKey() {

  const chars =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  let key = "";

  for (let i = 0; i < 16; i++) {

    key += chars[
      Math.floor(Math.random() * chars.length)
    ];

    if (i % 4 === 3 && i < 15)
      key += "-";

  }

  return key;

}

/* CREAR KEY */
window.createKey = async function () {

  try {

    const days = parseInt(
      document.getElementById("duration").value
    );

    const now = Date.now();

    const expiresAt =
      now + (days * 24 * 60 * 60 * 1000);

    const keyData = {

      key: genKey(),

      days: days,

      createdAt: now,

      expiresAt: expiresAt,

      used: false

    };

    await addDoc(
      collection(db, "keys"),
      keyData
    );

    document.getElementById("newKey").innerHTML =
      "KEY: <b>" + keyData.key + "</b>";

    loadKeys();

  } catch (err) {

    console.error(err);

    alert(err.message);

  }

};

/* CARGAR KEYS */
async function loadKeys() {

  const list =
    document.getElementById("list");

  list.innerHTML = "";

  const querySnapshot =
    await getDocs(
      collection(db, "keys")
    );

  querySnapshot.forEach(doc => {

    const k = doc.data();

    const div =
      document.createElement("div");

    div.className = "item";

    const expDate =
      new Date(k.expiresAt);

    const expired =
      Date.now() > k.expiresAt;

    div.innerHTML = `

      <b>${k.key}</b>

      <div class="small">
        Duración:
        ${k.days} días
      </div>

      <div class="small">
        Expira:
        ${expDate.toLocaleString()}
      </div>

      <div class="status">
        ${expired
          ? "❌ EXPIRADA"
          : "✅ ACTIVA"}
      </div>

      <div class="small">
        ${k.used
          ? "🔒 USADA"
          : "🟢 DISPONIBLE"}
      </div>

    `;

    list.appendChild(div);

  });

}

</script>

</body>
</html>
