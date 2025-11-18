<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Sistema Control IVA</title>
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
    rel="stylesheet"
  />
  <style>
    body {
      background: #f8f9fa;
      padding: 20px;
    }
    .card {
      border-radius: 15px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    .table td, .table th {
      vertical-align: middle;
    }
    .btn-sm {
      border-radius: 20px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2 class="text-center mb-4">📋 Sistema de Control de Clientes IVA</h2>

    <!-- FORMULARIO -->
    <div class="card mb-4">
      <div class="card-body">
        <h5>Registrar Cliente</h5>
        <form id="formCliente">
          <div class="row">
            <div class="col-md-4 mb-3">
              <label>Nombre / Razón Social</label>
              <input type="text" id="nombre" class="form-control" required />
            </div>
            <div class="col-md-4 mb-3">
              <label>NIT</label>
              <input type="text" id="nit" class="form-control" required />
            </div>
            <div class="col-md-4 mb-3">
              <label>NRC</label>
              <input type="text" id="nrc" class="form-control" />
            </div>
            <div class="col-md-4 mb-3">
              <label>Giro</label>
              <input type="text" id="giro" class="form-control" />
            </div>
            <div class="col-md-4 mb-3">
              <label>Correo</label>
              <input type="email" id="correo" class="form-control" />
            </div>
            <div class="col-md-4 mb-3">
              <label>Teléfono</label>
              <input type="text" id="telefono" class="form-control" />
            </div>
            <div class="col-md-4 mb-3">
              <label>Régimen</label>
              <select id="regimen" class="form-select">
                <option value="Común">Común</option>
                <option value="Pequeño contribuyente">Pequeño contribuyente</option>
              </select>
            </div>
            <div class="col-md-4 mb-3">
              <label>Fecha inscripción IVA</label>
              <input type="date" id="fechaInscripcion" class="form-control" />
            </div>
            <div class="col-md-4 mb-3">
              <label>Estado</label>
              <select id="estado" class="form-select">
                <option value="Activo">Activo</option>
                <option value="Inactivo">Inactivo</option>
              </select>
            </div>
          </div>
          <button type="submit" class="btn btn-primary mt-2">Guardar Cliente</button>
          <button type="reset" class="btn btn-secondary mt-2">Limpiar</button>
        </form>
      </div>
    </div>

    <!-- LISTADO -->
    <div class="card">
      <div class="card-body">
        <h5>Listado de Clientes</h5>
        <table class="table table-striped table-hover mt-3">
          <thead class="table-dark">
            <tr>
              <th>#</th>
              <th>Nombre</th>
              <th>NIT</th>
              <th>NRC</th>
              <th>Régimen</th>
              <th>Estado</th>
              <th>Dec.</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody id="tablaClientes"></tbody>
        </table>
      </div>
    </div>

    <!-- MÓDULO NUEVO: DECLARACIONES -->
    <div class="card mt-4">
      <div class="card-body">
        <h5>Control de Declaraciones IVA</h5>

        <!-- Selección de cliente -->
        <div class="row mb-3">
          <div class="col-md-6">
            <label>Cliente</label>
            <select id="clienteDeclaracion" class="form-select">
              <!-- Se llena por JS -->
            </select>
          </div>
        </div>

        <!-- Formulario de declaración -->
        <form id="formDeclaracion" class="mb-3">
          <div class="row">
            <div class="col-md-3 mb-3">
              <label>Período</label>
              <input type="month" id="periodo" class="form-control" required />
            </div>
            <div class="col-md-3 mb-3">
              <label>Fecha presentación</label>
              <input type="date" id="fechaPresentacion" class="form-control" />
            </div>
            <div class="col-md-3 mb-3">
              <label>Monto IVA</label>
              <input type="number" step="0.01" id="montoIVA" class="form-control" />
            </div>
            <div class="col-md-3 mb-3">
              <label>Estado</label>
              <select id="estadoDeclaracion" class="form-select">
                <option value="Pendiente">Pendiente</option>
                <option value="Presentada">Presentada</option>
                <option value="Vencida">Vencida</option>
              </select>
            </div>
            <div class="col-12 mb-3">
              <label>Nota</label>
              <textarea id="nota" class="form-control" rows="2"></textarea>
            </div>
          </div>
          <button type="submit" class="btn btn-primary btn-sm">Agregar declaración</button>
          <button type="reset" class="btn btn-secondary btn-sm">Limpiar</button>
        </form>

        <!-- Tabla de declaraciones por cliente -->
        <table class="table table-striped table-hover">
          <thead class="table-light">
            <tr>
              <th>#</th>
              <th>Período</th>
              <th>Fecha presentación</th>
              <th>Monto IVA</th>
              <th>Estado</th>
              <th>Nota</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody id="tablaDeclaraciones"></tbody>
        </table>
      </div>
    </div>

  </div>

  <script>
    // ========================
    //   CLIENTES (ORIGINAL)
    // ========================
    let clientes = JSON.parse(localStorage.getItem("clientesIVA")) || [];

    // Asegurar que todos tengan el arreglo de declaraciones
    clientes = clientes.map(c => ({
      ...c,
      declaraciones: c.declaraciones || []
    }));

    function guardarClientes() {
      localStorage.setItem("clientesIVA", JSON.stringify(clientes));
    }

    function mostrarClientes() {
      const tabla = document.getElementById("tablaClientes");
      tabla.innerHTML = "";
      clientes.forEach((c, i) => {
        const numDec = c.declaraciones ? c.declaraciones.length : 0;
        tabla.innerHTML += `
          <tr>
            <td>${i + 1}</td>
            <td>${c.nombre}</td>
            <td>${c.nit}</td>
            <td>${c.nrc || "-"}</td>
            <td>${c.regimen}</td>
            <td>${c.estado}</td>
            <td>${numDec}</td>
            <td>
              <button class="btn btn-sm btn-warning" onclick="editarCliente(${i})">Editar</button>
              <button class="btn btn-sm btn-danger" onclick="eliminarCliente(${i})">Eliminar</button>
            </td>
          </tr>`;
      });
    }

    document.getElementById("formCliente").addEventListener("submit", (e) => {
      e.preventDefault();
      const cliente = {
        nombre: document.getElementById("nombre").value,
        nit: document.getElementById("nit").value,
        nrc: document.getElementById("nrc").value,
        giro: document.getElementById("giro").value,
        correo: document.getElementById("correo").value,
        telefono: document.getElementById("telefono").value,
        regimen: document.getElementById("regimen").value,
        fechaInscripcion: document.getElementById("fechaInscripcion").value,
        estado: document.getElementById("estado").value,
        declaraciones: [] // nuevo campo para cada cliente
      };
      clientes.push(cliente);
      guardarClientes();
      mostrarClientes();
      actualizarSelectClientes();
      e.target.reset();
    });

    function eliminarCliente(i) {
      if (confirm("¿Eliminar este cliente?")) {
        clientes.splice(i, 1);
        guardarClientes();
        mostrarClientes();
        actualizarSelectClientes();
        mostrarDeclaracionesCliente(); // limpia si no hay cliente
      }
    }

    // Mantengo la lógica original: al editar, se rellena y se elimina el registro
    function editarCliente(i) {
      const c = clientes[i];
      document.getElementById("nombre").value = c.nombre;
      document.getElementById("nit").value = c.nit;
      document.getElementById("nrc").value = c.nrc;
      document.getElementById("giro").value = c.giro;
      document.getElementById("correo").value = c.correo;
      document.getElementById("telefono").value = c.telefono;
      document.getElementById("regimen").value = c.regimen;
      document.getElementById("fechaInscripcion").value = c.fechaInscripcion;
      document.getElementById("estado").value = c.estado;
      // OJO: aquí se pierde también sus declaraciones, igual que en el original
      eliminarCliente(i);
    }

    // ========================
    //   DECLARACIONES (NUEVO)
    // ========================
    const selectCliente = document.getElementById("clienteDeclaracion");
    const tablaDeclaraciones = document.getElementById("tablaDeclaraciones");

    function actualizarSelectClientes() {
      selectCliente.innerHTML = "";
      if (clientes.length === 0) {
        const opt = document.createElement("option");
        opt.value = "";
        opt.textContent = "No hay clientes registrados";
        selectCliente.appendChild(opt);
        return;
      }

      clientes.forEach((c, i) => {
        const opt = document.createElement("option");
        opt.value = i; // índice del cliente
        opt.textContent = `${c.nombre} - ${c.nit}`;
        selectCliente.appendChild(opt);
      });
    }

    function mostrarDeclaracionesCliente() {
      tablaDeclaraciones.innerHTML = "";
      const indice = selectCliente.value;
      if (indice === "" || clientes.length === 0) {
        return;
      }

      const cliente = clientes[indice];
      const declaraciones = cliente.declaraciones || [];

      declaraciones.forEach((d, i) => {
        tablaDeclaraciones.innerHTML += `
          <tr>
            <td>${i + 1}</td>
            <td>${d.periodo || ""}</td>
            <td>${d.fechaPresentacion || ""}</td>
            <td>${d.montoIVA || ""}</td>
            <td>${d.estado || ""}</td>
            <td>${d.nota || ""}</td>
            <td>
              <button class="btn btn-sm btn-danger" onclick="eliminarDeclaracion(${i})">
                Eliminar
              </button>
            </td>
          </tr>
        `;
      });
    }

    document.getElementById("formDeclaracion").addEventListener("submit", (e) => {
      e.preventDefault();
      const indice = selectCliente.value;
      if (indice === "" || clientes.length === 0) {
        alert("Primero selecciona un cliente.");
        return;
      }

      const declaracion = {
        periodo: document.getElementById("periodo").value,
        fechaPresentacion: document.getElementById("fechaPresentacion").value,
        montoIVA: document.getElementById("montoIVA").value,
        estado: document.getElementById("estadoDeclaracion").value,
        nota: document.getElementById("nota").value
      };

      const cliente = clientes[indice];
      if (!cliente.declaraciones) {
        cliente.declaraciones = [];
      }
      cliente.declaraciones.push(declaracion);

      guardarClientes();
      mostrarDeclaracionesCliente();
      mostrarClientes(); // para actualizar la columna Dec.
      e.target.reset();
    });

    function eliminarDeclaracion(indiceDec) {
      const indiceCliente = selectCliente.value;
      if (indiceCliente === "" || clientes.length === 0) return;

      if (confirm("¿Eliminar esta declaración?")) {
        const cliente = clientes[indiceCliente];
        cliente.declaraciones.splice(indiceDec, 1);
        guardarClientes();
        mostrarDeclaracionesCliente();
        mostrarClientes();
      }
    }

    selectCliente.addEventListener("change", mostrarDeclaracionesCliente);

    // ========================
    //   INICIO
    // ========================
    mostrarClientes();
    actualizarSelectClientes();
    mostrarDeclaracionesCliente();
  </script>
</body>
</html>
