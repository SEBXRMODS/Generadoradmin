<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Admin PRO</title>

<style>
body{margin:0;background:#000;color:#fff;font-family:Arial}
.container{padding:20px}
.card{background:#111;padding:15px;border-radius:10px;margin-bottom:10px}
button{margin:5px;padding:6px 10px;border:none;border-radius:6px;background:#3b82f6;color:#fff;cursor:pointer}
input{padding:8px;margin:5px;background:#222;border:none;color:#fff;border-radius:6px}
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

<h2>Panel Admin</h2>

<button id="logoutBtn">Cerrar sesión</button>

<h3>Crear Key</h3>
<input id="days" type="number" placeholder="Días">
<button id="createKey">Crear</button>

<h3>Keys</h3>
<div id="keys"></div>

<h3>Online Users</h3>
<div id="online"></div>

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
const onlineDiv = document.getElementById("online");

const createKeyBtn = document.getElementById("createKey");
const daysInput = document.getElementById("days");

/* LOGIN */
loginBtn.onclick = async ()=>{
try{
await signInWithEmailAndPassword(auth,email.value,password.value);
}catch(e){
loginStatus.innerText=e.message;
}
};

logoutBtn.onclick = async ()=>{
await signOut(auth);
};

/* SESSION */
onAuthStateChanged(auth,user=>{
if(user){
loginDiv.classList.add("hidden");
panel.classList.remove("hidden");
loadKeys();
loadOnline();
}else{
loginDiv.classList.remove("hidden");
panel.classList.add("hidden");
}
});

/* GENERAR KEY */
function genKey(){
return Math.random().toString(36).substring(2,6).toUpperCase()+"-"+
Math.random().toString(36).substring(2,6).toUpperCase()+"-"+
Math.random().toString(36).substring(2,6).toUpperCase();
}

/* CREAR KEY */
createKeyBtn.onclick = ()=>{
let days=parseInt(daysInput.value)||1;
let key=genKey();

let now=Date.now();
let exp=now+(days*86400000);

set(ref(db,"publicKeys/"+key),{
key,
createdAt:now,
expiresAt:exp,
days,
active:true,
used:false,
usedBy:"",
shared:false
});
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
Activa: ${k.active}<br>
Usada: ${k.used}<br>

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

/* FUNCIONES GLOBAL */
window.del = (key)=>{
remove(ref(db,"publicKeys/"+key));
};

window.toggle = (key,state)=>{
update(ref(db,"publicKeys/"+key),{active:!state});
};

window.add = async (key,d)=>{
let r=ref(db,"publicKeys/"+key);
onValue(r,s=>{
let v=s.val();
let newExp=v.expiresAt+(d*86400000);
update(r,{expiresAt:newExp});
},{onlyOnce:true});
};

/* ONLINE USERS */
function loadOnline(){
onValue(ref(db,"onlineUsers"),snap=>{
onlineDiv.innerHTML="";
snap.forEach(c=>{
let d=c.val();
let div=document.createElement("div");
div.className="card";
div.innerHTML=`${c.key} → ${d.key}`;
onlineDiv.appendChild(div);
});
});
}
</script>

</body>
</html>
