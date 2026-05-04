<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>App PRO</title>

<style>
body{margin:0;background:#000;color:#fff;font-family:Arial;overflow:hidden}
.center{height:100vh;display:flex;justify-content:center;align-items:center;flex-direction:column}
.box{background:#111;padding:25px;border-radius:10px;width:300px;text-align:center}
input{width:100%;padding:10px;margin-top:10px;background:#222;border:none;border-radius:6px;color:white}
button{width:100%;padding:10px;margin-top:10px;border:none;border-radius:6px;background:#3b82f6;color:white}
#particles{position:fixed;width:100%;height:100%;z-index:-1}
</style>
</head>

<body>

<canvas id="particles"></canvas>

<!-- LOGIN -->
<div id="login" class="center">
<div class="box">
<h2>Login</h2>
<input id="email" type="email" placeholder="Correo">
<input id="password" type="password" placeholder="Contraseña">
<button id="loginBtn">Entrar</button>
<p id="loginStatus"></p>
</div>
</div>

<!-- KEY -->
<div id="keyScreen" class="center" style="display:none">
<div class="box">
<h2>Key</h2>
<input id="keyInput" placeholder="XXXX-XXXX">
<button id="keyBtn">Validar</button>
<p id="keyStatus"></p>
</div>
</div>

<!-- APP -->
<div id="app" class="center" style="display:none">
<h2>Acceso OK</h2>
<p id="info"></p>
<button id="logoutBtn">Cerrar sesión</button>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";
import { getDatabase, ref, get, update, set, onValue, onDisconnect } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

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

/* DOM */
const loginDiv = document.getElementById("login");
const keyDiv = document.getElementById("keyScreen");
const appDiv = document.getElementById("app");

const email = document.getElementById("email");
const password = document.getElementById("password");
const keyInput = document.getElementById("keyInput");

const loginBtn = document.getElementById("loginBtn");
const keyBtn = document.getElementById("keyBtn");
const logoutBtn = document.getElementById("logoutBtn");

const loginStatus = document.getElementById("loginStatus");
const keyStatus = document.getElementById("keyStatus");
const info = document.getElementById("info");

/* DEVICE */
let device = localStorage.getItem("deviceId") || "DEV-"+Math.random().toString(36).substring(2,10);
localStorage.setItem("deviceId",device);

let unsub=null;

/* LOGIN AUTH */
loginBtn.onclick = async ()=>{
loginStatus.innerText="Entrando...";
try{
await signInWithEmailAndPassword(auth,email.value,password.value);
}catch(e){
loginStatus.innerText=e.message;
}
};

/* SESION */
onAuthStateChanged(auth,user=>{
if(user){
loginDiv.style.display="none";
keyDiv.style.display="flex";

/* auto key */
let saved = localStorage.getItem("savedKey");
if(saved) validateKey(saved);

}else{
loginDiv.style.display="flex";
keyDiv.style.display="none";
appDiv.style.display="none";
}
});

/* VALIDAR KEY */
async function validateKey(k=null){

let key=(k||keyInput.value).toUpperCase();
keyStatus.innerText="Verificando...";

let snap=await get(ref(db,"publicKeys/"+key));

if(!snap.exists()){keyStatus.innerText="❌ invalida";return;}
let d=snap.val();

if(!d.active){keyStatus.innerText="❌ desactivada";return;}
if(Date.now()>d.expiresAt){keyStatus.innerText="⏳ expirada";return;}

/* anti-share */
if(d.used && d.usedBy!==device){
await update(ref(db,"publicKeys/"+key),{shared:true});
keyStatus.innerText="🚫 compartida";return;
}

/* guardar uso */
await update(ref(db,"publicKeys/"+key),{
used:true,
usedBy:device
});

/* logs */
await set(ref(db,"logs/"+Date.now()),{
key,device,time:Date.now()
});

/* online */
let o=ref(db,"onlineUsers/"+device);
await set(o,{key,time:Date.now()});
onDisconnect(o).remove();

/* guardar local */
localStorage.setItem("savedKey",key);

/* realtime */
listen(key);

/* UI */
keyDiv.style.display="none";
appDiv.style.display="flex";
info.innerText="Key: "+key;
}

/* TIEMPO REAL */
function listen(key){
if(unsub)unsub();

unsub=onValue(ref(db,"publicKeys/"+key),(s)=>{
if(!s.exists()) logout("Key eliminada");

let d=s.val();

if(!d.active) logout("Key desactivada");
if(Date.now()>d.expiresAt) logout("Expirada");
if(d.used && d.usedBy!==device) logout("Otro dispositivo");
});
}

/* LOGOUT */
function logout(msg){
alert(msg);
localStorage.clear();
location.reload();
}

keyBtn.onclick=()=>validateKey();

logoutBtn.onclick=async()=>{
await signOut(auth);
localStorage.clear();
location.reload();
};
</script>

<!-- PARTICULAS -->
<script>
let c=document.getElementById("particles"),x=c.getContext("2d");
function r(){c.width=innerWidth;c.height=innerHeight}r();addEventListener("resize",r);
let p=[...Array(100)].map(()=>({x:Math.random()*c.width,y:Math.random()*c.height,vx:(Math.random()-0.5)*1.2,vy:(Math.random()-0.5)*1.2}));
function d(){
x.clearRect(0,0,c.width,c.height);
p.forEach(a=>{
a.x+=a.vx;a.y+=a.vy;
if(a.x<0||a.x>c.width)a.vx*=-1;
if(a.y<0||a.y>c.height)a.vy*=-1;
x.fillRect(a.x,a.y,2,2);
p.forEach(b=>{
let dist=Math.hypot(a.x-b.x,a.y-b.y);
if(dist<120){
x.strokeStyle="rgba(255,255,255,"+(1-dist/120)+")";
x.beginPath();x.moveTo(a.x,a.y);x.lineTo(b.x,b.y);x.stroke();
}
});
});
requestAnimationFrame(d);
}
d();
</script>

</body>
</html>
