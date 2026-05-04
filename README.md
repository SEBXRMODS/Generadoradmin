<!DOCTYPE html>
<html lang="es">

<head>

<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Panel Admin SaaS</title>

<style>

body{
margin:0;
background:#0b1220;
font-family:Arial;
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
width:330px;
box-shadow:0 0 30px rgba(0,0,0,.4);
}

input,
select{
width:100%;
padding:12px;
margin-top:10px;
border:none;
border-radius:6px;
box-sizing:border-box;
background:#0f172a;
color:white;
}

button{
width:100%;
padding:12px;
margin-top:10px;
border:none;
border-radius:6px;
background:#3b82f6;
color:white;
cursor:pointer;
font-weight:bold;
transition:.2s;
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

.card{
background:#111a2e;
padding:15px;
border-radius:10px;
margin-top:10px;
box-shadow:0 0 20px rgba(0,0,0,.2);
}

.grid{
display:grid;
grid-template-columns:
repeat(auto-fit,minmax(200px,1fr));
gap:15px;
margin-top:20px;
}

.key-item{
background:#0f172a;
padding:12px;
border-radius:8px;
margin-top:10px;
}

.small{
font-size:12px;
opacity:.7;
}

.shared{
border:2px solid red;
}

.actions{
display:grid;
gap:8px;
margin-top:10px;
}

h2,h3{
margin:0;
margin-bottom:10px;
}

</style>

</head>

<body>

<!-- LOGIN -->

<div id="login">

<div class="box">

<h2>
🔐 Admin Login
</h2>

<input
id="email"
type="email"
placeholder="Correo">

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

<!-- DASHBOARD -->

<div id="dash">

<h2 id="welcome"></h2>

<button id="logoutBtn">
Cerrar sesión
</button>

<div class="grid">

<div class="card">
<h3 id="totalKeys">0</h3>
<p>🔑 Keys</p>
</div>

<div class="card">
<h3 id="onlineUsers">0</h3>
<p>🟢 Online</p>
</div>

<div class="card">
<h3 id="sharedKeys">0</h3>
<p>🚫 Compartidas</p>
</div>

</div>

<!-- GENERAR -->

<div class="card">

<h3>
Generar Key
</h3>

<select id="duration">

<option value="1">
1 día
</option>

<option value="3">
3 días
</option>

<option value="7">
7 días
</option>

<option value="30">
30 días
</option>

<option value="365">
365 días
</option>

</select>

<button id="createKeyBtn">
Generar Key
</button>

<p id="newKey"></p>

</div>

<!-- KEYS -->

<div class="card">

<h3>
Mis Keys
</h3>

<div id="keysList"></div>

</div>

<!-- BAN -->

<div class="card">

<h3>
Banear Dispositivo
</h3>

<input
id="banDevice"
placeholder="DEV-XXXX">

<button id="banBtn">
Banear
</button>

</div>

<!-- LOGS -->

<div class="card">

<h3>
Logs
</h3>

<div id="logs"></div>

</div>

</div>

<script type="module">

import { initializeApp }

from

"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {

getAuth,
signInWithEmailAndPassword,
onAuthStateChanged,
signOut

}

from

"https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {

getDatabase,
ref,
set,
get,
child,
update

}

from

"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* FIREBASE */

const firebaseConfig = {

apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

authDomain: "panelsebxrmods.firebaseapp.com",

databaseURL: "https://panelsebxrmods-default-rtdb.firebaseio.com",

projectId: "panelsebxrmods",

storageBucket: "panelsebxrmods.firebasestorage.app",

messagingSenderId: "717339227525",

appId: "1:717339227525:web:98101a11654e25a45800ec"

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

document.getElementById(
"error"
).innerHTML =
err.message;

}

}

/* LOGOUT */

async function logout(){

await signOut(auth);

document.getElementById(
"dash"
).style.display =
"none";

document.getElementById(
"login"
).style.display =
"flex";

}

/* AUTH */

onAuthStateChanged(
auth,
async(user)=>{

if(user){

currentUID =
user.uid;

document.getElementById(
"login"
).style.display =
"none";

document.getElementById(
"dash"
).style.display =
"block";

const name =
user.email.split("@")[0];

document.getElementById(
"welcome"
).innerHTML =
"👋 Bienvenido " + name;

loadKeys();

loadStats();

loadLogs();

}

});

/* GENERATE KEY */

function generateKey(){

const chars =
"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

let result = "";

for(let i=0;i<16;i++){

result += chars[
Math.floor(
Math.random() *
chars.length
)
];

if(
i % 4 === 3 &&
i < 15
){

result += "-";

}

}

return result;

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

const key =
generateKey();

await set(

ref(
db,
"users/" +
currentUID +
"/keys/" +
key
),

{

key:key,
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
"✅ " + key;

loadKeys();

loadStats();

}

/* TOGGLE KEY */

async function toggleKey(key, active){

await update(

ref(
db,
"users/" +
currentUID +
"/keys/" +
key
),

{
active: !active
}

);

loadKeys();

}

/* ADD TIME */

async function addTime(key, days){

const keyRef =
ref(
db,
"users/" +
currentUID +
"/keys/" +
key
);

const snapshot =
await get(keyRef);

if(!snapshot.exists()) return;

const data =
snapshot.val();

const newExpire =
data.expiresAt +
(days * 86400000);

await update(

keyRef,

{
expiresAt:newExpire
}

);

loadKeys();

}

/* RESET DEVICE */

async function resetDevice(key){

await update(

ref(
db,
"users/" +
currentUID +
"/keys/" +
key
),

{

used:false,
usedBy:"",
shared:false

}

);

loadKeys();

}

/* DELETE KEY */

async function deleteKey(key){

const confirmDelete =
confirm(
"¿Eliminar esta key?"
);

if(!confirmDelete) return;

await set(

ref(
db,
"users/" +
currentUID +
"/keys/" +
key
),

null

);

loadKeys();

loadStats();

}

/* LOAD KEYS */

async function loadKeys(){

const list =
document.getElementById(
"keysList"
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
"key-item";

if(k.shared){

div.classList.add(
"shared"
);

}

const exp =
new Date(
k.expiresAt
);

div.innerHTML = `

<b>${k.key}</b>

<div class="small">
⏳ ${exp.toLocaleString()}
</div>

<div class="small">
📱 ${k.usedBy || "Sin usar"}
</div>

<div class="small">

${
k.shared
? "🚫 Compartida"
: k.used
? "🔒 En uso"
: "🟢 Disponible"
}

</div>

<div class="small">

Estado:
${
k.active
? "🟢 Activa"
: "❌ Desactivada"
}

</div>

<div class="actions">

<button
onclick="toggleKey('${k.key}', ${k.active})">

${
k.active
? "❌ Desactivar"
: "✅ Activar"
}

</button>

<button
onclick="addTime('${k.key}', 1)">
+1 Día
</button>

<button
onclick="addTime('${k.key}', 3)">
+3 Días
</button>

<button
onclick="addTime('${k.key}', 7)">
+7 Días
</button>

<button
onclick="resetDevice('${k.key}')">
🔄 Reset Device
</button>

<button
onclick="deleteKey('${k.key}')">
🗑️ Eliminar
</button>

</div>

`;

list.appendChild(div);

});

}

}

/* STATS */

async function loadStats(){

let total = 0;
let shared = 0;

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

if(k.shared){

shared++;

}

});

}

document.getElementById(
"totalKeys"
).innerHTML =
total;

document.getElementById(
"sharedKeys"
).innerHTML =
shared;

const onlineSnap =
await get(
child(ref(db),
"onlineUsers")
);

if(onlineSnap.exists()){

document.getElementById(
"onlineUsers"
).innerHTML =

Object.keys(
onlineSnap.val()
).length;

}

}

/* LOAD LOGS */

async function loadLogs(){

const logs =
document.getElementById(
"logs"
);

logs.innerHTML = "";

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
"key-item";

div.innerHTML = `

📱 ${log.device}

<div class="small">
🔑 ${log.key}
</div>

`;

logs.appendChild(div);

});

}

}

/* BAN DEVICE */

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
"🚫 Dispositivo baneado"
);

}

/* BUTTONS */

window.onload = () => {

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

};

/* GLOBAL */

window.toggleKey =
toggleKey;

window.addTime =
addTime;

window.resetDevice =
resetDevice;

window.deleteKey =
deleteKey;

</script>

</body>
</html>
