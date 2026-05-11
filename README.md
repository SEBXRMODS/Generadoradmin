<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Panel Admin Keys</title>

<style>

body{
background:#0a0a0a;
color:white;
font-family:Arial;
padding:20px;
margin:0;
}

h1{
text-align:center;
margin-bottom:30px;
}

.key{
background:#111;
border:1px solid #333;
padding:20px;
border-radius:15px;
margin-bottom:20px;
}

button{
padding:10px 15px;
border:none;
border-radius:10px;
background:#00ff88;
color:black;
font-weight:bold;
cursor:pointer;
margin:5px;
}

.danger{
background:red;
color:white;
}

input{
width:100%;
padding:12px;
margin-bottom:15px;
background:#111;
border:1px solid #444;
color:white;
border-radius:10px;
}

#panel{
display:none;
}

</style>
</head>
<body>

<div id="loginBox">

<h1>ADMIN LOGIN</h1>

<input type="email" id="email" placeholder="Correo admin">

<input type="password" id="password" placeholder="Contraseña">

<button onclick="login()">
Entrar
</button>

<p id="error"></p>

</div>

<div id="panel">

<h1>PANEL ADMIN KEYS</h1>

<input
type="text"
id="buscar"
placeholder="Buscar key o email..."
oninput="filtrarKeys()">

<div id="keysContainer"></div>

</div>

<script type="module">

import { initializeApp }
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
getAuth,
signInWithEmailAndPassword,
onAuthStateChanged
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {
getFirestore,
collection,
getDocs,
doc,
updateDoc,
deleteDoc,
getDoc
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

const container =
document.getElementById(
"keysContainer"
);

let todasLasKeys = [];

// UID ADMIN

const admins = [

"PEGA_AQUI_TU_UID"

];

// LOGIN

window.login = async function(){

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
).innerHTML = err.message;

}

}

// VERIFICAR ADMIN

onAuthStateChanged(
auth,
async(user)=>{

if(!user)return;

if(!admins.includes(user.uid)){

alert("NO ERES ADMIN");

return;

}

document.getElementById(
"loginBox"
).style.display = "none";

document.getElementById(
"panel"
).style.display = "block";

cargarKeys();

});

// CARGAR KEYS

async function cargarKeys(){

container.innerHTML = "";

const querySnapshot =
await getDocs(
collection(db,"keys")
);

todasLasKeys = [];

querySnapshot.forEach((docSnap)=>{

const data =
docSnap.data();

todasLasKeys.push({

id:docSnap.id,

...data

});

});

mostrarKeys(todasLasKeys);

}

// MOSTRAR KEYS

function mostrarKeys(lista){

container.innerHTML = "";

lista.forEach((data)=>{

container.innerHTML += `

<div class="key">

<h2>${data.key}</h2>

<p>
<b>Producto:</b>
${data.producto}
</p>

<p>
<b>Duración:</b>
${data.duracion}
</p>

<p>
<b>Estado:</b>
${data.estado}
</p>

<p>
<b>Email:</b>
${data.email}
</p>

<p>
<b>UID:</b>
${data.uid}
</p>

<p>
<b>Expira:</b>
${data.expira}
</p>

<button
onclick="añadirTiempo('${data.id}',7)">

+7 Días

</button>

<button
onclick="añadirTiempo('${data.id}',30)">

+1 Mes

</button>

<button
onclick="añadirTiempo('${data.id}',365)">

+1 Año

</button>

<button
onclick="desactivarKey('${data.id}')">

Desactivar

</button>

<button
class="danger"
onclick="eliminarKey('${data.id}')">

Eliminar

</button>

</div>

`;

});

}

// FILTRAR

window.filtrarKeys = function(){

const texto =
document.getElementById(
"buscar"
).value.toLowerCase();

const filtradas =
todasLasKeys.filter((k)=>{

return (

(k.key || "")
.toLowerCase()
.includes(texto)

||

(k.email || "")
.toLowerCase()
.includes(texto)

);

});

mostrarKeys(filtradas);

}

// AÑADIR TIEMPO

window.añadirTiempo =
async function(id,dias){

const ref =
doc(db,"keys",id);

const snap =
await getDoc(ref);

const data =
snap.data();

const actual =
new Date(data.expira);

actual.setDate(
actual.getDate()+dias
);

await updateDoc(ref,{

expira:
actual.toISOString()

});

alert("TIEMPO AÑADIDO");

cargarKeys();

}

// DESACTIVAR

window.desactivarKey =
async function(id){

await updateDoc(
doc(db,"keys",id),
{

estado:"desactivada"

}

);

cargarKeys();

}

// ELIMINAR

window.eliminarKey =
async function(id){

const confirmar =
confirm(
"¿Eliminar key?"
);

if(!confirmar)return;

await deleteDoc(
doc(db,"keys",id)
);

cargarKeys();

}

</script>

</body>
</html>
