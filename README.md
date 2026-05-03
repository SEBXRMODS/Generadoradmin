<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Panel SaaS</title>

<style>

body{
  margin:0;
  font-family:Arial;
  background:#0b1220;
  color:white;
}

#login{
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}

.box{
  background:#111a2e;
  padding:30px;
  border-radius:12px;
  width:320px;
}

input,select{
  width:100%;
  padding:10px;
  margin-top:10px;
  border:none;
  border-radius:6px;
  box-sizing:border-box;
}

button{
  width:100%;
  padding:10px;
  margin-top:10px;
  border:none;
  border-radius:6px;
  background:#3b82f6;
  color:white;
  cursor:pointer;
}

#dash{
  display:none;
  padding:20px;
}

.panel{
  display:grid;
  grid-template-columns:1fr 2fr;
  gap:20px;
}

.section{
  background:#111a2e;
  padding:15px;
  border-radius:10px;
}

.item{
  background:#0f172a;
  padding:10px;
  border-radius:6px;
  margin-top:10px;
}

.small{
  opacity:.7;
  font-size:12px;
}

</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">

  <div class="box">

    <h2>Admin Login</h2>

    <input
      id="user"
      placeholder="Usuario">

    <input
      id="pass"
      type="password"
      placeholder="Contraseña">

    <button onclick="login()">
      Entrar
    </button>

    <p id="error"></p>

  </div>

</div>

<!-- PANEL -->
<div id="dash">

  <h2>Panel SaaS</h2>

  <button onclick="logout()">
    Cerrar sesión
  </button>

  <br><br>

  <div class="panel">

    <!-- GENERAR -->
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
        Generar Key
      </button>

      <p id="newKey"></p>

    </div>

    <!-- LISTA -->
    <div class="section">

      <h3>Keys</h3>

      <div id="list"></div>

    </div>

  </div>

</div>

<script type="module">

import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getDatabase,
  ref,
  set,
  get,
  child
} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* FIREBASE */
const firebaseConfig = {

  apiKey:
  "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain:
  "panelsebxrmods.firebaseapp.com",

  databaseURL:
  "https://panelsebxrmods-default-rtdb.firebaseio.com",

  projectId:
  "panelsebxrmods",

  storageBucket:
  "panelsebxrmods.firebasestorage.app",

  messagingSenderId:
  "717339227525",

  appId:
  "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

const app = initializeApp(firebaseConfig);

const db = getDatabase(app);

/* LOGIN SIMPLE */
const ADMIN_USER = "admin";
const ADMIN_PASS = "1234";

window.login = function(){

  const u =
    document.getElementById("user").value;

  const p =
    document.getElementById("pass").value;

  if(
    u === ADMIN_USER &&
    p === ADMIN_PASS
  ){

    document.getElementById(
      "login"
    ).style.display = "none";

    document.getElementById(
      "dash"
    ).style.display = "block";

    loadKeys();

  }else{

    document.getElementById(
      "error"
    ).innerHTML =
      "Datos incorrectos";

  }

};

/* LOGOUT */
window.logout = function(){

  location.reload();

};

/* GENERADOR */
function genKey(){

  const chars =
  "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  let key = "";

  for(let i=0;i<16;i++){

    key += chars[
      Math.floor(
        Math.random() * chars.length
      )
    ];

    if(i % 4 === 3 && i < 15){

      key += "-";

    }

  }

  return key;

}

/* CREAR KEY */
window.createKey = async function(){

  try{

    const days = parseInt(

      document.getElementById(
        "duration"
      ).value

    );

    const now = Date.now();

    const expiresAt =

      now +
      (
        days *
        24 *
        60 *
        60 *
        1000
      );

    const newKey = genKey();

    const keyData = {

      key:newKey,

      days:days,

      createdAt:now,

      expiresAt:expiresAt,

      used:false,

      active:true

    };

    await set(

      ref(
        db,
        "keys/" + newKey
      ),

      keyData

    );

    document.getElementById(
      "newKey"
    ).innerHTML =

      "<b>" +
      newKey +
      "</b>";

    loadKeys();

  }catch(err){

    console.error(err);

    alert(err.message);

  }

};

/* CARGAR KEYS */
async function loadKeys(){

  const list =
    document.getElementById("list");

  list.innerHTML = "";

  const snapshot = await get(

    child(
      ref(db),
      "keys"
    )

  );

  if(snapshot.exists()){

    const data = snapshot.val();

    Object.values(data)
    .reverse()
    .forEach(k => {

      const exp =
        new Date(k.expiresAt);

      const expired =
        Date.now() >
        k.expiresAt;

      const div =
        document.createElement("div");

      div.className = "item";

      div.innerHTML = `

        <b>${k.key}</b>

        <div class="small">
          ${k.days} días
        </div>

        <div class="small">
          Expira:
          ${exp.toLocaleString()}
        </div>

        <div class="small">
          ${
            expired
            ? "❌ Expirada"
            : "✅ Activa"
          }
        </div>

      `;

      list.appendChild(div);

    });

  }

}

</script>

</body>
</html>
