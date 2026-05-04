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
<input id="email" placeholder="Correo">
<input id="password" type="password" placeholder="Contraseña">
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
import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

/* CONFIG */
const firebaseConfig = {
apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",
authDomain: "panelsebxrmods.firebaseapp.com",
projectId: "panelsebxrmods",
storageBucket: "panelsebxrmods.appspot.com",
messagingSenderId: "717339227525",
appId: "1:717339227525:web:98101a11654e25a45800ec"
};

const appFirebase = initializeApp(firebaseConfig);
const auth = getAuth(appFirebase);

/* ELEMENTOS */
const loginDiv = document.getElementById("login");
const appDiv = document.getElementById("app");

const emailInput = document.getElementById("email");
const passwordInput = document.getElementById("password");
const loginBtn = document.getElementById("loginBtn");
const logoutBtn = document.getElementById("logoutBtn");
const status = document.getElementById("status");

/* LOGIN */
loginBtn.onclick = async ()=>{
status.innerText="Entrando...";

try{
await signInWithEmailAndPassword(
auth,
emailInput.value,
passwordInput.value
);
}catch(e){
status.innerText = e.message;
}
};

/* SESIÓN */
onAuthStateChanged(auth, user=>{
if(user){
loginDiv.style.display="none";
appDiv.style.display="flex";
}else{
loginDiv.style.display="flex";
appDiv.style.display="none";
}
});

/* LOGOUT */
logoutBtn.onclick = async ()=>{
await signOut(auth);
location.reload();
};

</script>

<!-- PARTICULAS PRO -->
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
