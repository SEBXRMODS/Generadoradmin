<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Admin Sebxr Mods</title>

<style>

body{
background:#0a0a0a;
color:white;
font-family:Arial;
padding:20px;
margin:0;
}

h1,h2{
text-align:center;
}

.card{
background:#111;
border:1px solid #333;
padding:20px;
border-radius:15px;
margin-bottom:20px;
}

input{
width:100%;
padding:12px;
margin-bottom:10px;
background:#111;
color:white;
border:1px solid #444;
border-radius:10px;
}

button{
padding:12px 20px;
border:none;
border-radius:10px;
background:#00ff88;
color:black;
font-weight:bold;
cursor:pointer;
margin:5px;
}

.userCard{
background:#151515;
border:1px solid #333;
padding:15px;
border-radius:15px;
margin-top:20px;
word-break:break-word;
}

.info{
margin-bottom:10px;
}

</style>
</head>
<body>

<!-- LOGIN -->

<div id="loginBox" class="card">

<h1>
ADMIN PANEL
</h1>

<input
type="email"
id="email"
placeholder="Correo admin">

<input
type="password"
id="password"
placeholder="Contraseña">

<button onclick="loginAdmin()">
ENTRAR
</button>

<p id="error"></p>

</div>

<!-- PANEL -->

<div id="adminPanel" style="display:none;">

<h1>
🔥 PANEL ADMIN SEBXR MODS
</h1>

<div class="card">

<h2>
Buscar usuario
</h2>

<input
type="text"
id="buscarInput"
placeholder="UID o correo">

<button onclick="buscarUsuario()">
BUSCAR
</button>

</div>

<div id="resultadoUsuario"></div>

</div>

<script type="module">

import { initializeApp }
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
getAuth,
signInWithEmailAndPassword
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {
getFirestore,
doc,
getDoc,
updateDoc,
collection,
getDocs,
deleteDoc,
query,
where
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

// FIREBASE

const firebaseConfig = {

apiKey: "AIzaSyBtbovWtH-fnSA2KqbobIFjtbNtcicsi-k",

authDomain:
"pagina-de-productos-680db.firebaseapp.com",

projectId:
"pagina-de-productos-680db",

storageBucket:
"pagina-de-productos-680db.firebasestorage.app",

messagingSenderId:
"393545047716",

appId:
"1:393545047716:web:07fb512bc7ee970bdbd031",

measurementId:
"G-EZBWVZDK5E"

};

const app =
initializeApp(firebaseConfig);

const auth =
getAuth(app);

const db =
getFirestore(app);

// LOGIN ADMIN

window.loginAdmin =
async function(){

const emailValue =
email.value;

const passwordValue =
password.value;

try{

const cred =
await signInWithEmailAndPassword(
auth,
emailValue,
passwordValue
);

const uid =
cred.user.uid;

// VERIFICAR ADMIN

const adminRef =
doc(db,"admins",uid);

const adminSnap =
await getDoc(adminRef);

if(!adminSnap.exists()){

alert(
"NO ERES ADMIN"
);

return;

}

loginBox.style.display =
"none";

adminPanel.style.display =
"block";

}catch(err){

console.log(err);

error.innerHTML =
err.message;

}

}

// BUSCAR USUARIO

window.buscarUsuario =
async function(){

const valor =
buscarInput.value
.toLowerCase();

resultadoUsuario.innerHTML =
"Buscando...";

const usersRef =
collection(db,"users");

const snap =
await getDocs(usersRef);

let encontrado = false;

resultadoUsuario.innerHTML =
"";

for(const docu of snap.docs){

const data =
docu.data();

const uid =
docu.id;

const correo =
(data.email || "")
.toLowerCase();

if(
uid.includes(valor) ||
correo.includes(valor)
){

encontrado = true;

// KEYS

const keysQuery =
query(
collection(db,"keys"),
where("uid","==",uid)
);

const keysSnap =
await getDocs(keysQuery);

let htmlKeys = "";

keysSnap.forEach((k)=>{

const kd =
k.data();

htmlKeys += `

<div class="card">

<b>Producto:</b>
${kd.producto}

<br><br>

<b>Key:</b>
${kd.key}

<br><br>

<b>Duración:</b>
${kd.duracion}

<br><br>

<b>Estado:</b>
${kd.estado}

<br><br>

<button onclick="desactivarKey('${k.id}')">
DESACTIVAR
</button>

<button onclick="activarKey('${k.id}')">
ACTIVAR
</button>

<button onclick="eliminarKey('${k.id}')">
ELIMINAR
</button>

</div>

`;

});

if(htmlKeys == ""){

htmlKeys =
"<p>Sin keys</p>";

}

// HISTORIAL

const comprasQuery =
query(
collection(db,"purchases"),
where("uid","==",uid)
);

const comprasSnap =
await getDocs(comprasQuery);

let htmlCompras = "";

comprasSnap.forEach((c)=>{

const cd =
c.data();

htmlCompras += `

<div class="card">

<b>${cd.producto}</b>

<br><br>

💰 ${cd.creditos}
 créditos

<br><br>

📅 ${cd.fecha}

</div>

`;

});

if(htmlCompras == ""){

htmlCompras =
"<p>Sin historial</p>";

}

// HTML

resultadoUsuario.innerHTML += `

<div class="userCard">

<h2>
👤 Usuario
</h2>

<div class="info">

<b>Email:</b><br>

${data.email}

</div>

<div class="info">

<b>UID:</b><br>

${uid}

</div>

<div class="info">

<b>Créditos:</b><br>

<span id="creditos-${uid}">
${data.creditos || 0}
</span>

</div>

<button onclick="sumarCreditos('${uid}',100)">
+100
</button>

<button onclick="sumarCreditos('${uid}',500)">
+500
</button>

<button onclick="sumarCreditos('${uid}',1000)">
+1000
</button>

<br>

<button onclick="restarCreditos('${uid}',100)">
-100
</button>

<button onclick="restarCreditos('${uid}',500)">
-500
</button>

<h2>
🔑 Keys
</h2>

${htmlKeys}

<h2>
🧾 Historial
</h2>

${htmlCompras}

</div>

`;

}

}

if(!encontrado){

resultadoUsuario.innerHTML =

`

<div class="card">

Usuario no encontrado

</div>

`;

}

}

// SUMAR CREDITOS

window.sumarCreditos =
async function(uid,cantidad){

const ref =
doc(db,"users",uid);

const snap =
await getDoc(ref);

const data =
snap.data();

const actuales =
Number(data.creditos)||0;

const nuevos =
actuales + cantidad;

await updateDoc(ref,{

creditos:nuevos

});

document.getElementById(
`creditos-${uid}`
).innerHTML =
nuevos;

alert(
"CRÉDITOS AÑADIDOS"
);

}

// RESTAR CREDITOS

window.restarCreditos =
async function(uid,cantidad){

const ref =
doc(db,"users",uid);

const snap =
await getDoc(ref);

const data =
snap.data();

const actuales =
Number(data.creditos)||0;

let nuevos =
actuales - cantidad;

if(nuevos < 0){

nuevos = 0;

}

await updateDoc(ref,{

creditos:nuevos

});

document.getElementById(
`creditos-${uid}`
).innerHTML =
nuevos;

alert(
"CRÉDITOS QUITADOS"
);

}

// DESACTIVAR KEY

window.desactivarKey =
async function(id){

await updateDoc(
doc(db,"keys",id),
{

estado:"desactivada"

});

alert(
"KEY DESACTIVADA"
);

}

// ACTIVAR KEY

window.activarKey =
async function(id){

await updateDoc(
doc(db,"keys",id),
{

estado:"activa"

});

alert(
"KEY ACTIVADA"
);

}

// ELIMINAR KEY

window.eliminarKey =
async function(id){

await deleteDoc(
doc(db,"keys",id)
);

alert(
"KEY ELIMINADA"
);

}

</script>

</body>
</html>
