<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Admin PRO</title>

<style>
body{margin:0;background:#000;color:#fff;font-family:Arial}
.container{padding:20px}
.card{background:#111;padding:10px;border-radius:8px;margin:5px}
button{margin:3px;padding:6px;border:none;border-radius:6px;background:#3b82f6;color:white}
select,input{padding:6px;background:#222;color:#fff;border:none;border-radius:6px}
.hidden{display:none}
</style>
</head>

<body>

<div id="login" class="container">
<h2>Admin Login</h2>
<input id="email" placeholder="Correo">
<input id="password" type="password" placeholder="Contraseña">
<button id="loginBtn">Entrar</button>
<p id="loginStatus"></p>
</div>

<div id="panel" class="container hidden">

<button id="logoutBtn">Cerrar sesión</button>

<h3>Crear Key</h3>

<select id="timeSelect">
<option value="1">1 día</option>
<option value="2">2 días</option>
<option value="3">3 días</option>
<option value="4">4 días</option>
<option value="5">5 días</option>
<option value="6">6 días</option>
<option value="7">7 días</option>
<option value="30">1 mes</option>
<option value="365">1 año</option>
</select>

<button id="createKey">Generar</button>

<p id="createStatus"></p>

<h3>Keys</h3>
<div id="keys"></div>

</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";
import { getDatabase, ref, set, onValue, remove, update } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

const firebaseConfig = {
apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",
authDomain: "panelsebxrmods.firebaseapp.com",
databaseURL: "https://panelsebxrmods-default-rtdb.firebaseio.com",
projectId: "panelsebxrmods",
storageBucket: "panelsebxrmods.appspot.com",
messagingSenderId: "717339227525",
appId: "1:717339227525:web:98101a11654e25a45800ec"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getDatabase(app);

const loginDiv = document.getElementById("login");
const panel = document.getElementById("panel");

const loginBtn = document.getElementById("loginBtn");
const logoutBtn = document.getElementById("logoutBtn");

const email = document.getElementById("email");
const password = document.getElementById("password");
const loginStatus = document.getElementById("loginStatus");

const keysDiv = document.getElementById("keys");

const createKeyBtn = document.getElementById("createKey");
const timeSelect = document.getElementById("timeSelect");
const createStatus = document.getElementById("createStatus");

/* LOGIN */
loginBtn.onclick = async ()=>{
try{
await signInWithEmailAndPassword(auth,email.value,password.value);
}catch(e){
loginStatus.innerText=e.message;
}
};

logoutBtn.onclick = ()=>signOut(auth);

/* SESSION */
onAuthStateChanged(auth,user=>{
if(user){
loginDiv.classList.add("hidden");
panel.classList.remove("hidden");
loadKeys();
}else{
loginDiv.classList.remove("hidden");
panel.classList.add("hidden");
}
});

/* GENERAR KEY REAL */
function genKey(){
const chars="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
let k="";
for(let i=0;i<16;i++){
k+=chars[Math.floor(Math.random()*chars.length)];
if((i+1)%4===0 && i!==15) k+="-";
}
return k;
}

/* CREAR KEY (FIX) */
createKeyBtn.onclick = async ()=>{

let days = parseInt(timeSelect.value);
if(!days){
createStatus.innerText="Selecciona tiempo";
return;
}

let key = genKey();
let now = Date.now();
let exp = now + (days * 86400000);

await set(ref(db,"publicKeys/"+key),{
key,
createdAt:now,
expiresAt:exp,
days,
active:true,
used:false,
usedBy:"",
shared:false
});

createStatus.innerText="✔ Key creada: "+key;
};

/* LISTAR KEYS */
function loadKeys(){
onValue(ref(db,"publicKeys"),snap=>{
keysDiv.innerHTML="";
snap.forEach(child=>{
let k=child.val();

let div=document.createElement("div");
div.className="card";

div.innerHTML=`
<b>${k.key}</b><br>
Activa: ${k.active} | Usada: ${k.used}<br>

<button onclick="toggle('${k.key}',${k.active})">ON/OFF</button>
<button onclick="del('${k.key}')">Eliminar</button>
<button onclick="add('${k.key}',1)">+1d</button>
<button onclick="add('${k.key}',3)">+3d</button>
<button onclick="add('${k.key}',7)">+7d</button>
`;

keysDiv.appendChild(div);
});
});
}

/* FUNCIONES */
window.del = key => remove(ref(db,"publicKeys/"+key));

window.toggle = (key,state)=>{
update(ref(db,"publicKeys/"+key),{active:!state});
};

window.add = (key,d)=>{
onValue(ref(db,"publicKeys/"+key),s=>{
let v=s.val();
update(ref(db,"publicKeys/"+key),{
expiresAt: v.expiresAt + (d*86400000)
});
},{onlyOnce:true});
};
</script>

</body>
</html>
