<script type="module">

import { initializeApp }
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getAuth,
  signInWithEmailAndPassword,
  onAuthStateChanged,
  signOut
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {
  getDatabase,
  ref,
  set,
  get,
  child
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* FIREBASE CONFIG */
const firebaseConfig = {

  apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain: "panelsebxrmods.firebaseapp.com",

  databaseURL:
    "https://panelsebxrmods-default-rtdb.firebaseio.com",

  projectId: "panelsebxrmods",

  storageBucket:
    "panelsebxrmods.firebasestorage.app",

  messagingSenderId: "717339227525",

  appId:
    "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

/* INIT */
const app = initializeApp(firebaseConfig);

const auth = getAuth(app);

const db = getDatabase(app);

/* LOGIN */
window.login = async function () {

  const email =
    document.getElementById("user").value;

  const password =
    document.getElementById("pass").value;

  try {

    await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

  } catch (err) {

    document.getElementById("error").innerText =
      err.message;

  }

};

/* SESSION */
onAuthStateChanged(auth, user => {

  if (user) {

    document.getElementById("login").style.display =
      "none";

    document.getElementById("dash").style.display =
      "block";

    loadKeys();

  } else {

    document.getElementById("login").style.display =
      "flex";

    document.getElementById("dash").style.display =
      "none";

  }

});

/* LOGOUT */
window.logout = async function () {

  await signOut(auth);

};

/* GENERADOR */
function genKey() {

  const chars =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  let key = "";

  for (let i = 0; i < 16; i++) {

    key += chars[
      Math.floor(Math.random() * chars.length)
    ];

    if (i % 4 === 3 && i < 15)
      key += "-";

  }

  return key;

}

/* CREAR KEY */
window.createKey = async function () {

  try {

    const days = parseInt(
      document.getElementById("duration").value
    );

    const now = Date.now();

    const expiresAt =
      now + (days * 24 * 60 * 60 * 1000);

    const newKey = genKey();

    const keyData = {

      key: newKey,

      days: days,

      createdAt: now,

      expiresAt: expiresAt,

      used: false,

      active: true

    };

    /* GUARDAR */
    await set(
      ref(db, "keys/" + newKey),
      keyData
    );

    document.getElementById("newKey").innerHTML =
      "KEY: <b>" + newKey + "</b>";

    loadKeys();

  } catch (err) {

    alert(err.message);

  }

};

/* CARGAR KEYS */
async function loadKeys() {

  const list =
    document.getElementById("list");

  list.innerHTML = "";

  const dbRef = ref(db);

  const snapshot = await get(
    child(dbRef, "keys")
  );

  if (snapshot.exists()) {

    const data = snapshot.val();

    Object.values(data).forEach(k => {

      const div =
        document.createElement("div");

      div.className = "item";

      const exp =
        new Date(k.expiresAt);

      const expired =
        Date.now() > k.expiresAt;

      div.innerHTML = `

        <b>${k.key}</b>

        <div class="small">
          ${k.days} días
        </div>

        <div class="small">
          Expira:
          ${exp.toLocaleString()}
        </div>

        <div class="status">
          ${expired
            ? "❌ EXPIRADA"
            : "✅ ACTIVA"}
        </div>

      `;

      list.appendChild(div);

    });

  }

}

</script>
