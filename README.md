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

button:hover{
  background:#2563eb;
}

#error{
  color:red;
  margin-top:10px;
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

.stats-grid{
  display:grid;
  grid-template-columns:
  repeat(auto-fit,minmax(160px,1fr));
  gap:15px;
  margin-bottom:20px;
}

.stat-card{
  background:#111a2e;
  padding:20px;
  border-radius:12px;
  text-align:center;
}

.stat-card h3{
  margin:0;
  font-size:28px;
  color:#3b82f6;
}

</style>
</head>

<body>

<div id="login">

<div class="box">

<h2>Admin Login</h2>

<input
id="email"
type="email"
placeholder="Email">

<input
id="password"
type="password"
placeholder="Contraseña">

<button id="loginBtn">
Entrar
</button>

<p id="error"></p>

</div>

</div>

<div id="dash">

<h2>Panel SaaS</h2>

<h3 id="welcomeAdmin"></h3>

<div class="stats-grid">

<div class="stat-card">
<h3 id="totalKeys">0</h3>
<p>🔑 Totales</p>
</div>

<div class="stat-card">
<h3 id="activeKeys">0</h3>
<p>✅ Activas</p>
</div>

<div class="stat-card">
<h3 id="expiredKeys">0</h3>
<p>❌ Expiradas</p>
</div>

<div class="stat-card">
<h3 id="usedKeys">0</h3>
<p>🔒 Usadas</p>
</div>

<div class="stat-card">
<h3 id="onlineUsersStat">0</h3>
<p>🟢 Online</p>
</div>

<div class="stat-card">
<h3 id="bannedCount">0</h3>
<p>🚫 Baneados</p>
</div>

</div>

<button id="logoutBtn">
Cerrar sesión
</button>

<br><br>

<div class="panel">

<div class="section">

<h3>Crear Key</h3>

<select id="duration">

<option value="1">1 día</option>
<option value="7">7 días</option>
<option value="30">30 días</option>
<option value="365">1 año</option>

</select>

<button id="createKeyBtn">
Generar Key
</button>

<p id="newKey"></p>

</div>

<div class="section">

<h3>Mis Keys</h3>

<div id="list"></div>

</div>

</div>

<br>

<div class="section">

<h3>Banear dispositivo</h3>

<input
id="banDevice"
placeholder="device-id">

<button id="banBtn">
Banear
</button>

</div>

<br>

<div class="section">

<h3>Logs</h3>

<div id="logs"></div>

</div>

</div>

<script type="module">

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
child

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

const firebaseConfig = {

apiKey:"TU_API_KEY",

authDomain:"TU_AUTH_DOMAIN",

databaseURL:"TU_DATABASE_URL",

projectId:"TU_PROJECT_ID",

storageBucket:"TU_STORAGE_BUCKET",

messagingSenderId:"TU_SENDER_ID",

appId:"TU_APP_ID"

};

const app =
initializeApp(firebaseConfig);

const auth =
getAuth(app);

const db =
getDatabase(app);

let currentUID = "";

/* LOGIN */

async function login(){

const email =
document.getElementById(
"email"
).value;

const password =
document.getElementById(
"password"
).value;

try{

await signInWithEmailAndPassword(
auth,
email,
password
);

}catch(err){

console.log(err);

document.getElementById(
"error"
).innerHTML =
err.message;

}

}

/* LOGOUT */

async function logout(){

await signOut(auth);

}

/* SESSION */

onAuthStateChanged(
auth,
(user)=>{

const loginDiv =
document.getElementById(
"login"
);

const dashDiv =
document.getElementById(
"dash"
);

if(user){

currentUID = user.uid;

const emailName =
user.email
.split("@")[0];

document.getElementById(
"welcomeAdmin"
).innerHTML =
"Bienvenido, " +
emailName;

loginDiv.style.display =
"none";

dashDiv.style.display =
"block";

loadKeys();

loadStats();

loadLogs();

}else{

loginDiv.style.display =
"flex";

dashDiv.style.display =
"none";

}

});

/* GENERAR KEY */

function genKey(){

const chars =
"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

let key = "";

for(let i=0;i<16;i++){

key += chars[
Math.floor(
Math.random() *
chars.length
)
];

if(
i % 4 === 3 &&
i < 15
){

key += "-";

}

}

return key;

}

/* CREATE KEY */

async function createKey(){

const days =
parseInt(
document.getElementById(
"duration"
).value
);

const now =
Date.now();

const expiresAt =
now +
(days * 86400000);

const newKey =
genKey();

await set(

ref(

db,

"users/" +
currentUID +
"/keys/" +
newKey

),

{

key:newKey,

days:days,

createdAt:now,

expiresAt:expiresAt,

used:false,

usedBy:"",

active:true,

shared:false

}

);

document.getElementById(
"newKey"
).innerHTML =
newKey;

loadKeys();

loadStats();

}

/* LOAD KEYS */

async function loadKeys(){

const list =
document.getElementById(
"list"
);

list.innerHTML = "";

const snapshot =
await get(

child(

ref(db),

"users/" +
currentUID +
"/keys"

)

);

if(snapshot.exists()){

const data =
snapshot.val();

Object.values(data)
.reverse()
.forEach(k=>{

const div =
document.createElement(
"div"
);

div.className =
"item";

if(k.shared){

div.style.border =
"2px solid red";

}

const exp =
new Date(
k.expiresAt
);

div.innerHTML = `

<b>${k.key}</b>

<div class="small">
${k.days} días
</div>

<div class="small">
⏳ Expira:
${exp.toLocaleString()}
</div>

<div class="small">
📱 Device:
${k.usedBy || "Sin usar"}
</div>

<div class="small">

Estado:

${
k.shared
? "🚫 COMPARTIDA"
: k.used
? "🔒 En uso"
: "🟢 Disponible"
}

</div>

`;

list.appendChild(div);

});

}

}

/* STATS */

async function loadStats(){

let total = 0;
let active = 0;
let expired = 0;
let used = 0;

const snapshot =
await get(

child(

ref(db),

"users/" +
currentUID +
"/keys"

)

);

if(snapshot.exists()){

const data =
snapshot.val();

Object.values(data)
.forEach(k=>{

total++;

if(
Date.now() >
k.expiresAt
){

expired++;

}else{

active++;

}

if(k.used){

used++;

}

});

}

document.getElementById(
"totalKeys"
).innerHTML =
total;

document.getElementById(
"activeKeys"
).innerHTML =
active;

document.getElementById(
"expiredKeys"
).innerHTML =
expired;

document.getElementById(
"usedKeys"
).innerHTML =
used;

}

async function loadLogs(){

const logsDiv =
document.getElementById(
"logs"
);

logsDiv.innerHTML = "";

const snapshot =
await get(
child(ref(db),
"logs")
);

if(snapshot.exists()){

const data =
snapshot.val();

Object.values(data)
.reverse()
.forEach(log=>{

const div =
document.createElement(
"div"
);

div.className =
"item";

div.innerHTML = `

📱 ${log.device}

<div class="small">
🔑 ${log.key}
</div>

`;

logsDiv.appendChild(div);

});

}

}

async function banDevice(){

const device =
document.getElementById(
"banDevice"
).value;

await set(

ref(

db,

"bannedDevices/" +
device

),

true

);

alert(
"Dispositivo baneado"
);

}

/* BUTTON EVENTS */

document
.getElementById(
"loginBtn"
)
.addEventListener(
"click",
login
);

document
.getElementById(
"logoutBtn"
)
.addEventListener(
"click",
logout
);

document
.getElementById(
"createKeyBtn"
)
.addEventListener(
"click",
createKey
);

document
.getElementById(
"banBtn"
)
.addEventListener(
"click",
banDevice
);

</script>

</body>
</html>
