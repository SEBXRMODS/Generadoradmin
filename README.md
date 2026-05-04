<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>App</title>

<style>
body{
margin:0;
font-family:Arial;
color:white;
overflow:hidden;
}

/* PARTICULAS */
#particles{
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
z-index:-1;
background:black;
}

/* UI */
.center{
height:100vh;
display:flex;
justify-content:center;
align-items:center;
flex-direction:column;
}

.box{
background:#111;
padding:30px;
border-radius:10px;
width:300px;
text-align:center;
}

input{
width:100%;
padding:10px;
margin-top:10px;
border:none;
border-radius:6px;
background:#222;
color:white;
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
</style>
</head>

<body>

<canvas id="particles"></canvas>

<!-- LOGIN -->
<div id="login" class="center">
<div class="box">
<h2>Login</h2>
<input id="keyInput" placeholder="Ingresa tu key">
<button id="loginBtn">Entrar</button>
<p id="status"></p>
</div>
</div>

<!-- APP -->
<div id="app" class="center" style="display:none">
<h2>✅ Bienvenido</h2>
<button id="logoutBtn">Cerrar sesión</button>
</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getDatabase, ref, get, update, set, onValue } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* 🔥 CONFIG */
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
const db = getDatabase(app);

/* DEVICE */
if(!localStorage.getItem("deviceId")){
localStorage.setItem("deviceId","DEV-"+Math.random().toString(36).substring(2,10));
}
const device = localStorage.getItem("deviceId");

let unsubscribe = null;

/* AUTO LOGIN */
window.onload = async ()=>{
const savedKey = localStorage.getItem("savedKey");
if(savedKey){
validateKey(savedKey,true);
}
};

/* VALIDAR */
async function validateKey(inputKey=null,auto=false){

const key = inputKey || document.getElementById("keyInput").value;

status.innerText="Verificando...";

try{

const snap = await get(ref(db,"publicKeys/"+key));

if(!snap.exists()){
status.innerText="❌ Key inválida";
localStorage.removeItem("savedKey");
return;
}

const data = snap.val();

/* checks */
if(!data.active){
status.innerText="❌ Desactivada";
return;
}

if(Date.now()>data.expiresAt){
status.innerText="⏳ Expirada";
return;
}

/* anti-share */
if(data.used && data.usedBy !== device){

await update(ref(db,"publicKeys/"+key),{shared:true});

status.innerText="🚫 Compartida";
return;
}

/* marcar uso */
await update(ref(db,"publicKeys/"+key),{
used:true,
usedBy:device
});

/* log */
await set(ref(db,"logs/"+Date.now()),{
key,device,time:Date.now()
});

/* guardar */
localStorage.setItem("savedKey",key);

/* escuchar en tiempo real */
listenKey(key);

/* entrar */
enterApp();

}catch(e){
console.error(e);
status.innerText="Error conexión";
}

}

/* TIEMPO REAL */
function listenKey(key){

const keyRef = ref(db,"publicKeys/"+key);

unsubscribe = onValue(keyRef,(snap)=>{

if(!snap.exists()){
forceLogout("❌ Key eliminada");
return;
}

const data = snap.val();

if(!data.active){
forceLogout("🚫 Key desactivada");
return;
}

if(Date.now()>data.expiresAt){
forceLogout("⏳ Expirada");
return;
}

if(data.used && data.usedBy !== device){
forceLogout("🚫 Uso en otro dispositivo");
return;
}

});
}

/* EXPULSAR */
function forceLogout(msg){

alert(msg);

localStorage.removeItem("savedKey");

if(unsubscribe) unsubscribe();

location.reload();
}

/* ENTRAR */
function enterApp(){
document.getElementById("login").style.display="none";
document.getElementById("app").style.display="flex";
}

/* EVENTOS */
loginBtn.onclick=()=>validateKey();

logoutBtn.onclick=()=>{
localStorage.removeItem("savedKey");
location.reload();
};

</script>

<!-- 🔥 PARTICULAS PRO -->
<script>
const canvas=document.getElementById("particles");
const ctx=canvas.getContext("2d");

let particles=[];
const max=100;

function resize(){
canvas.width=innerWidth;
canvas.height=innerHeight;
}
resize();
addEventListener("resize",resize);

for(let i=0;i<max;i++){
particles.push({
x:Math.random()*canvas.width,
y:Math.random()*canvas.height,
vx:(Math.random()-0.5)*1.2,
vy:(Math.random()-0.5)*1.2,
size:Math.random()*2+1
});
}

function draw(){

ctx.clearRect(0,0,canvas.width,canvas.height);

particles.forEach((p,i)=>{

p.x+=p.vx;
p.y+=p.vy;

if(p.x<0||p.x>canvas.width)p.vx*=-1;
if(p.y<0||p.y>canvas.height)p.vy*=-1;

ctx.beginPath();
ctx.arc(p.x,p.y,p.size,0,Math.PI*2);
ctx.fillStyle="white";
ctx.fill();

for(let j=i+1;j<particles.length;j++){
const p2=particles[j];

const dx=p.x-p2.x;
const dy=p.y-p2.y;
const dist=Math.sqrt(dx*dx+dy*dy);

if(dist<120){
ctx.beginPath();
ctx.moveTo(p.x,p.y);
ctx.lineTo(p2.x,p2.y);
ctx.strokeStyle=`rgba(255,255,255,${1-dist/120})`;
ctx.stroke();
}
}

});

requestAnimationFrame(draw);
}

draw();
</script>

</body>
</html>
