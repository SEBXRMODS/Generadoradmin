<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<title>Firebase Test</title>
</head>

<body style="background:#0b1220;color:white;font-family:Arial;">

<h1>Firebase funcionando</h1>

<script type="module">

import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

const firebaseConfig = {

  apiKey:
  "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain:
  "panelsebxrmods.firebaseapp.com",

  databaseURL:
  "https://panelsebxrmods-default-rtdb.firebaseio.com",

  projectId:
  "panelsebxrmods",

  storageBucket:
  "panelsebxrmods.firebasestorage.app",

  messagingSenderId:
  "717339227525",

  appId:
  "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

const app = initializeApp(firebaseConfig);

console.log("firebase iniciado");

document.body.innerHTML +=
"<h2>Firebase conectado ✅</h2>";

</script>

</body>
</html>
