<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Panel Admin</title>

<style>
body{margin:0;background:#0b1220;font-family:Arial;color:white}
#login{height:100vh;display:flex;justify-content:center;align-items:center}
.box{background:#111a2e;padding:30px;border-radius:12px;width:330px}
input,select{width:100%;padding:12px;margin-top:10px;border:none;border-radius:6px;background:#0f172a;color:white}
button{width:100%;padding:12px;margin-top:10px;border:none;border-radius:6px;background:#3b82f6;color:white;cursor:pointer}
#dash{display:none;padding:20px}
.card{background:#111a2e;padding:15px;border-radius:10px;margin-top:10px}
.key-item{background:#0f172a;padding:12px;border-radius:8px;margin-top:10px}
.actions{display:grid;gap:6px;margin-top:10px}
</style>
</head>

<body>

<div id="login">
<div class="box">
<h2>Admin Login</h2>
<input id="email" placeholder="Correo">
<input id="password" type="password" placeholder="Contraseña">
<button id="loginBtn">Entrar</button>
<p id="error"></p>
</div>
</div>

<div id="dash">

<h2 id="welcome"></h2>
<button id="logoutBtn">Cerrar sesión</button>

<div class="card">
<h3>Generar Key</h3>

<select id="duration">
<option value="1">1 día</option>
<option value="3">3 días</option>
<option value="7">7 días</option>
<option value="30">30 días</option>
</select>

<button id="createKeyBtn">Generar</button>
<p id="newKey"></p>

</div>

<div class="card">
<h3>Keys</h3>
<div id="keysList"></div>
</div>

</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";
import { getDatabase, ref, set, get, update, child } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

const firebaseConfig = {
apiKey: "TU_API_KEY",
authDomain: "TU_AUTH",
databaseURL: "TU_DB",
projectId: "TU_ID",
storageBucket: "TU_BUCKET",
messagingSenderId: "TU_MSG",
appId: "TU_APP"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getDatabase(app);

let currentUID = "";

/* LOGIN */
async function login(){
const email = emailInput.value;
const pass = passwordInput.value;

try{
await signInWithEmailAndPassword(auth,email,pass);
}catch(e){
error.innerText = e.message;
}
}

/* LOGOUT */
async function logout(){
await signOut(auth);
location.reload();
}

/* AUTH */
onAuthStateChanged(auth, async user=>{
if(user){

currentUID = user.uid;

loginDiv.style.display="none";
dash.style.display="block";

welcome.innerText = "Bienvenido " + user.email.split("@")[0];

loadKeys();

}
});

/* GENERAR KEY */
function generateKey(){
const chars="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
let key="";
for(let i=0;i<16;i++){
key+=chars[Math.floor(Math.random()*chars.length)];
if(i%4===3 && i<15) key+="-";
}
return key;
}

/* CREAR KEY */
async function createKey(){

const days = parseInt(duration.value);
const now = Date.now();
const expiresAt = now + days*86400000;
const key = generateKey();

/* PRIVADO */
await set(ref(db,"users/"+currentUID+"/keys/"+key),{
key,days,createdAt:now,expiresAt,
used:false,usedBy:"",active:true,shared:false
});

/* PUBLICO */
await set(ref(db,"publicKeys/"+key),{
uid:currentUID,
expiresAt,
active:true,
used:false,
usedBy:""
});

newKey.innerText = key;

loadKeys();
}

/* TOGGLE */
async function toggleKey(key,active){

await update(ref(db,"users/"+currentUID+"/keys/"+key),{
active:!active
});

await update(ref(db,"publicKeys/"+key),{
active:!active
});

loadKeys();
}

/* ADD TIME */
async function addTime(key,days){

const snap = await get(ref(db,"users/"+currentUID+"/keys/"+key));
if(!snap.exists()) return;

const data = snap.val();
const newExpire = data.expiresAt + days*86400000;

await update(ref(db,"users/"+currentUID+"/keys/"+key),{
expiresAt:newExpire
});

await update(ref(db,"publicKeys/"+key),{
expiresAt:newExpire
});

loadKeys();
}

/* DELETE */
async function deleteKey(key){

if(!confirm("Eliminar key?")) return;

await set(ref(db,"users/"+currentUID+"/keys/"+key),null);
await set(ref(db,"publicKeys/"+key),null);

loadKeys();
}

/* LOAD KEYS */
async function loadKeys(){

keysList.innerHTML="";

const snap = await get(child(ref(db),"users/"+currentUID+"/keys"));

if(!snap.exists()) return;

Object.values(snap.val()).reverse().forEach(k=>{

const div = document.createElement("div");
div.className="key-item";

div.innerHTML=`
<b>${k.key}</b>
<div>Expira: ${new Date(k.expiresAt).toLocaleString()}</div>
<div>Uso: ${k.usedBy||"Libre"}</div>
<div>Estado: ${k.active?"Activa":"Desactivada"}</div>

<div class="actions">
<button onclick="toggleKey('${k.key}',${k.active})">Toggle</button>
<button onclick="addTime('${k.key}',1)">+1d</button>
<button onclick="addTime('${k.key}',3)">+3d</button>
<button onclick="resetDevice('${k.key}')">Reset</button>
<button onclick="deleteKey('${k.key}')">Eliminar</button>
</div>
`;

keysList.appendChild(div);

});
}

/* RESET */
async function resetDevice(key){

await update(ref(db,"users/"+currentUID+"/keys/"+key),{
used:false,usedBy:"",shared:false
});

await update(ref(db,"publicKeys/"+key),{
used:false,usedBy:""
});

loadKeys();
}

/* EVENTS */
loginBtn.onclick=login;
logoutBtn.onclick=logout;
createKeyBtn.onclick=createKey;

/* GLOBAL */
window.toggleKey=toggleKey;
window.addTime=addTime;
window.deleteKey=deleteKey;
window.resetDevice=resetDevice;

</script>

</body>
</html>
