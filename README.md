<script type="module">

import { initializeApp }
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getFirestore,
  collection,
  addDoc,
  getDocs
}
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

/* FIREBASE */
const firebaseConfig = {

  apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain: "panelsebxrmods.firebaseapp.com",

  projectId: "panelsebxrmods",

  storageBucket: "panelsebxrmods.firebasestorage.app",

  messagingSenderId: "717339227525",

  appId: "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

const app = initializeApp(firebaseConfig);

const db = getFirestore(app);

/* LOGIN */
window.login = function () {

  const u =
    document.getElementById("user").value;

  const p =
    document.getElementById("pass").value;

  if (u === "admin" && p === "1234") {

    document.getElementById("login").style.display =
      "none";

    document.getElementById("dash").style.display =
      "block";

    loadKeys();

  }

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

    const keyData = {

      key: genKey(),

      days: days,

      createdAt: now,

      expiresAt: expiresAt,

      used: false

    };

    await addDoc(
      collection(db, "keys"),
      keyData
    );

    document.getElementById("newKey").innerHTML =
      "KEY: <b>" + keyData.key + "</b>";

    loadKeys();

  } catch (err) {

    console.error(err);

    alert(err.message);

  }

};

/* CARGAR KEYS */
async function loadKeys() {

  const list =
    document.getElementById("list");

  list.innerHTML = "";

  const querySnapshot =
    await getDocs(
      collection(db, "keys")
    );

  querySnapshot.forEach(doc => {

    const k = doc.data();

    const div =
      document.createElement("div");

    div.className = "item";

    div.innerHTML = `

      <b>${k.key}</b>

      <div class="small">
        ${k.days} días
      </div>

    `;

    list.appendChild(div);

  });

}

</script>
