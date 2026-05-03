<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<title>Realtime Test</title>
</head>

<body style="background:#0b1220;color:white;font-family:Arial;">

<h1>Realtime Database Test</h1>

<button onclick="guardar()">
Guardar Key
</button>

<p id="estado"></p>

<script type="module">

import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getDatabase,
  ref,
  set
} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* CONFIG */
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

/* INIT */
const app = initializeApp(firebaseConfig);

const db = getDatabase(app);

/* BOTON */
window.guardar = async function () {

  try {

    await set(

      ref(db, "test/hola"),

      {
        mensaje: "funciona"
      }

    );

    document.getElementById("estado").innerHTML =
      "GUARDADO ✅";

    console.log("guardado");

  } catch (err) {

    console.error(err);

    alert(err.message);

  }

};

</script>

</body>
</html>
