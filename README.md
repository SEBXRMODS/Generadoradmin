<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Admin Keys</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>

<div class="container">
  <h2>Generar Key</h2>

  <select id="duracion">
    <option value="1">1 día</option>
    <option value="2">2 días</option>
    <option value="7">7 días</option>
    <option value="30">1 mes</option>
    <option value="365">1 año</option>
  </select>

  <button onclick="crear()">Generar</button>
  <p id="nuevaKey"></p>
</div>

<script type="module" src="admin.js"></script>
</body>
</html>
