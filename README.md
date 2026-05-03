<script type="module">

import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
getAuth,
signInWithEmailAndPassword,
onAuthStateChanged
} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

/* CONFIG */

const firebaseConfig = {

apiKey:"TU_API_KEY",

authDomain:"TU_AUTH_DOMAIN",

projectId:"TU_PROJECT_ID",

appId:"TU_APP_ID"

};

const app =
initializeApp(firebaseConfig);

const auth =
getAuth(app);

/* TEST */

console.log("Firebase iniciado");

/* LOGIN */

async function login(){

const email =
document.getElementById(
"email"
).value;

const password =
document.getElementById(
"password"
).value;

console.log("Intentando login");

try{

const userCred =

await signInWithEmailAndPassword(
auth,
email,
password
);

console.log(
"LOGIN OK",
userCred.user
);

}catch(err){

console.log(
"ERROR LOGIN:",
err
);

document.getElementById(
"error"
).innerHTML =
err.message;

}

}

document
.getElementById(
"loginBtn"
)
.addEventListener(
"click",
login
);

/* SESSION */

onAuthStateChanged(
auth,
(user)=>{

console.log(
"AUTH CHANGED",
user
);

if(user){

document.getElementById(
"login"
).style.display =
"none";

document.getElementById(
"dash"
).style.display =
"block";

}

});

</script>
