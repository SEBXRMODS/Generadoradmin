<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

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

button:hover{
  background:#2563eb;
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

.actions{
  display:flex;
  gap:10px;
  margin-top:10px;
}

.actions button{
  flex:1;
}

</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">

  <div class="box">

    <h2>Admin Login</h2>

    <input
      id="email"
      placeholder="Email">

    <input
      id="password"
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

  <h3 id="onlineCount">
    Usuarios online: 0
  </h3>

  <button onclick="logout()">
    Cerrar sesión
  </button>

  <button onclick="deleteExpired()">
    Borrar expiradas
  </button>

  <br><br>

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

      <button onclick="createKey()">
        Generar Key
      </button>

      <p id="newKey"></p>

    </div>

    <!-- KEYS -->
    <div class="section">

      <h3>Keys</h3>

      <div id="list"></div>

    </div>

  </div>

  <br>

  <!-- LOGS -->
  <div class="section">

    <h3>Logs Usuarios</h3>

    <div id="logs"></div>

  </div>

</div>

<script type="module">

/* FIREBASE */
import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {

  getAuth,
  signInWithEmailAndPassword,
  onAuthStateChanged,
  signOut

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {

  getDatabase,
  ref,
  set,
  get,
  child,
  remove,
  onValue

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* CONFIG */
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

/* INIT */
const app = initializeApp(firebaseConfig);

const auth = getAuth(app);

const db = getDatabase(app);

/* LOGIN */
window.login = async function(){

  const email =
    document.getElementById("email").value;

  const password =
    document.getElementById("password").value;

  try{

    await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

  }catch(err){

    document.getElementById(
      "error"
    ).innerHTML =
      err.message;

  }

};

/* SESSION */
onAuthStateChanged(auth, (user) => {

  const loginDiv =
    document.getElementById("login");

  const dashDiv =
    document.getElementById("dash");

  if(user){

    loginDiv.style.display = "none";

    dashDiv.style.display = "block";

    loadKeys();

    loadLogs();

  }else{

    loginDiv.style.display = "flex";

    dashDiv.style.display = "none";

  }

});

/* LOGOUT */
window.logout = async function(){

  await signOut(auth);

};

/* GENERAR KEY */
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

      usedBy:"",

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
      "<b>" + newKey + "</b>";

    loadKeys();

  }catch(err){

    alert(err.message);

  }

};

/* ACTIVAR / DESACTIVAR */
window.toggleKey = async function(key, current){

  await set(

    ref(
      db,
      "keys/" + key + "/active"
    ),

    !current

  );

  loadKeys();

}

/* RENOVAR */
window.renewKey = async function(key, extraDays){

  const snapshot = await get(
    child(ref(db), "keys/" + key)
  );

  if(snapshot.exists()){

    const data = snapshot.val();

    const newExpire =
      data.expiresAt +
      (
        extraDays *
        24 *
        60 *
        60 *
        1000
      );

    await set(
      ref(
        db,
        "keys/" + key + "/expiresAt"
      ),
      newExpire
    );

    loadKeys();

  }

}

/* BORRAR EXPIRADAS */
window.deleteExpired = async function(){

  const snapshot = await get(
    child(ref(db), "keys")
  );

  if(snapshot.exists()){

    const data = snapshot.val();

    for(const k of Object.values(data)){

      if(Date.now() > k.expiresAt){

        await remove(
          ref(
            db,
            "keys/" + k.key
          )
        );

      }

    }

  }

  loadKeys();

}

/* CARGAR KEYS */
async function loadKeys(){

  const list =
    document.getElementById("list");

  list.innerHTML = "";

  const snapshot = await get(
    child(ref(db), "keys")
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

      const remainingMs =
        k.expiresAt - Date.now();

      const remainingHours =
        Math.floor(
          remainingMs /
          (1000 * 60 * 60)
        );

      const remainingDays =
        Math.floor(
          remainingHours / 24
        );

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
          Tiempo restante:
          ${
            expired
            ? "0 horas"
            : remainingDays > 0
            ? remainingDays + " días"
            : remainingHours + " horas"
          }
        </div>

        <div class="small">
          Estado:
          ${
            expired
            ? "❌ Expirada"
            : k.active
            ? "✅ Activa"
            : "⛔ Desactivada"
          }
        </div>

        <div class="small">
          ${
            k.used
            ? "🔒 Usada por: " + k.usedBy
            : "🟢 Sin usar"
          }
        </div>

        <div class="actions">

          <button onclick="toggleKey('${k.key}', ${k.active})">
            ${k.active ? "Desactivar" : "Activar"}
          </button>

          <button onclick="renewKey('${k.key}', 30)">
            +30 días
          </button>

        </div>

      `;

      list.appendChild(div);

    });

  }

}

/* LOGS */
async function loadLogs(){

  const logsDiv =
    document.getElementById("logs");

  logsDiv.innerHTML = "";

  const snapshot = await get(
    child(ref(db), "logs")
  );

  if(snapshot.exists()){

    const data = snapshot.val();

    Object.values(data)
    .reverse()
    .forEach(log => {

      const div =
        document.createElement("div");

      div.className = "item";

      div.innerHTML = `

        <div>
          📱 ${log.device}
        </div>

        <div class="small">
          🔑 ${log.key}
        </div>

        <div class="small">
          🕒
          ${new Date(log.time).toLocaleString()}
        </div>

      `;

      logsDiv.appendChild(div);

    });

  }

}

/* AGREGAR LOG */
async function addLog(device, key){

  const id = Date.now();

  await set(

    ref(db, "logs/" + id),

    {

      device: device,
      key: key,
      time: Date.now()

    }

  );

}

/* AUTO ACTUALIZAR */
setInterval(() => {

  loadKeys();

}, 60000);

/* AUTO DELETE */
setInterval(async () => {

  const snapshot = await get(
    child(ref(db), "keys")
  );

  if(snapshot.exists()){

    const data = snapshot.val();

    for(const k of Object.values(data)){

      if(Date.now() > k.expiresAt){

        await remove(
          ref(db, "keys/" + k.key)
        );

      }

    }

  }

}, 60000);

/* ONLINE */
onValue(ref(db, "onlineUsers"), snapshot => {

  if(snapshot.exists()){

    const total =
      Object.keys(
        snapshot.val()
      ).length;

    document.getElementById(
      "onlineCount"
    ).innerHTML =
      "Usuarios online: " + total;

  }else{

    document.getElementById(
      "onlineCount"
    ).innerHTML =
      "Usuarios online: 0";

  }

});

</script>

</body>
</html>
