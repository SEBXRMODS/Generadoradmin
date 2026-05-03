<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Login Admin</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>

<div class="container">
  <h2>Login Admin</h2>
  <input type="email" id="email" placeholder="Correo">
  <input type="password" id="password" placeholder="Contraseña">
  <button onclick="login()">Entrar</button>
  <p id="error"></p>
</div>

<script type="module" src="auth.js"></script>
</body>
</html>
