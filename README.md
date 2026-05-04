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
background:black;
color:white;
overflow:hidden;
}

/* fondo animado */
.bg{
position:fixed;
width:100%;
height:100%;
z-index:-1;
background:linear-gradient(270deg,#ff00c8,#00ffe7,#ff8c00,#ff00c8);
background-size:800% 800%;
animation:moveBg 15s ease infinite;
filter:blur(100px);
opacity:0.5;
}

@keyframes moveBg{
0%{background-position:0% 50%}
50%{background-position:100% 50%}
100%{background-position:0% 50%}
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

<div class="bg"></div>

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
import { getDatabase, ref, get, update, set } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* CONFIG */
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

/* AUTO LOGIN */
window.onload = async () => {
const savedKey = localStorage.getItem("savedKey");
if(savedKey){
validateKey(savedKey,true);
}
};

/* VALIDAR */
async function validateKey(inputKey=null,auto=false){

const key = inputKey || document.getElementById("keyInput").value;

status.innerText = "Verificando...";

try{

const snap = await get(ref(db,"publicKeys/"+key));

if(!snap.exists()){
status.innerText = "❌ Key inválida";
localStorage.removeItem("savedKey");
return;
}

const data = snap.val();

/* checks */
if(!data.active){
status.innerText = "❌ Desactivada";
return;
}

if(Date.now() > data.expiresAt){
status.innerText = "⏳ Expirada";
return;
}

/* anti-share */
if(data.used && data.usedBy !== device){

await update(ref(db,"publicKeys/"+key),{shared:true});

status.innerText = "🚫 Compartida";
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

enterApp();

}catch(e){
console.error(e);
status.innerText = "Error conexión";
}

}

/* ENTRAR */
function enterApp(){
document.getElementById("login").style.display="none";
document.getElementById("app").style.display="flex";
}

/* EVENTOS */
loginBtn.onclick = () => validateKey();

logoutBtn.onclick = () => {
localStorage.removeItem("savedKey");
location.reload();
};

</script>

</body>
</html>
