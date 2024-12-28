<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema ERP MILI - Facturación</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        /* Estilos generales */
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #f5f7fa;
            color: #333;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #0066cc;
            color: white;
            text-align: center;
            padding: 15px 0;
        }
        h1 {
            margin: 0;
            font-size: 2.5em;
        }

        .container {
            width: 80%;
            margin: 20px auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        .form-section {
            margin-bottom: 30px;
        }

        .form-section h2 {
            font-size: 1.8em;
            color: #0066cc;
            margin-bottom: 10px;
        }

        label {
            font-size: 1em;
            color: #666;
            margin-bottom: 5px;
            display: block;
        }

        input[type="text"], input[type="number"], input[type="date"], select {
            width: 100%;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 8px;
            margin-bottom: 15px;
            font-size: 1em;
            color: #333;
            box-sizing: border-box;
        }

        input[type="button"] {
            background-color: #0066cc;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 8px;
            font-size: 1em;
            cursor: pointer;
            transition: background-color 0.3s;
            margin-top: 15px;
        }

        input[type="button"]:hover {
            background-color: #005bb5;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        table, th, td {
            border: 1px solid #ddd;
        }

        th {
            background-color: #0066cc;
            color: white;
            padding: 12px;
            text-align: left;
        }

        td {
            padding: 12px;
            text-align: left;
        }

        .total {
            font-weight: bold;
            text-align: right;
            padding-top: 20px;
        }

        .total .amount {
            color: #0066cc;
            font-size: 1.2em;
        }

        .listado-section {
            margin-top: 50px;
        }

        .listado-section h2 {
            font-size: 1.8em;
            color: #0066cc;
            margin-bottom: 10px;
        }

        .listado-table th, .listado-table td {
            text-align: center;
        }
    </style>
</head>
<body>

<header>
    <h1>Sistema ERP MILI - Facturación</h1>
</header>

<div class="container">
    <!-- Datos generales -->
    <div class="form-section">
        <h2>Datos de la Factura</h2>
        <form id="facturaForm">
            <label for="tipoComprobante">Tipo de Comprobante</label>
            <select id="tipoComprobante">
                <option value="Factura">Factura</option>
                <option value="Boleta">Boleta</option>
                <option value="Nota de Crédito">Nota de Crédito</option>
                <option value="Nota de Débito">Nota de Débito</option>
            </select>

            <label for="serie">Serie</label>
            <input type="text" id="serie" placeholder="Ingrese la serie" value="F001" required>

            <label for="numeracion">Numeración</label>
            <input type="text" id="numeracion" placeholder="Ingrese la numeración" value="000001" required>

            <label for="fecha">Fecha</label>
            <input type="text" id="fecha" readonly>

            <label for="fechaVencimiento">Fecha de Vencimiento</label>
            <input type="date" id="fechaVencimiento" value="2024-12-30">

            <label for="moneda">Tipo de Moneda</label>
            <select id="moneda" onchange="actualizarMoneda()">
                <option value="PEN">Nuevo Sol (PEN)</option>
                <option value="USD">Dólar (USD)</option>
            </select>

            <label for="direccionEmisor">Dirección del Emisor</label>
            <input type="text" id="direccionEmisor" placeholder="Ingrese la dirección del emisor" required>

            <label for="direccionReceptor">Dirección del Receptor</label>
            <input type="text" id="direccionReceptor" placeholder="Ingrese la dirección del receptor" required>
        </form>
    </div>

    <!-- Productos -->
    <div class="form-section">
        <h2>Productos</h2>
        <table id="productosTable">
            <thead>
                <tr>
                    <th>Descripción</th>
                    <th>Cantidad</th>
                    <th>Precio Unitario</th>
                    <th>Base Imponible</th>
                    <th>IGV</th>
                    <th>Importe Total</th>
                    <th>Acción</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><input type="text" class="descripcion" placeholder="Descripción del producto"></td>
                    <td><input type="number" class="cantidad" min="1" placeholder="Cantidad" onchange="calcularTotales()"></td>
                    <td><input type="number" class="precioUnitario" min="0" placeholder="Precio Unitario" onchange="calcularTotales()"></td>
                    <td><input type="number" class="baseImponible" min="0" placeholder="Base Imponible" readonly></td>
                    <td><input type="number" class="iva" min="0" placeholder="IGV" readonly></td>
                    <td><input type="number" class="importeTotal" min="0" placeholder="Importe Total" readonly></td>
                    <td><input type="button" class="eliminar" value="Eliminar" onclick="eliminarFila(this)"></td>
                </tr>
            </tbody>
        </table>

        <input type="button" value="Agregar Producto" onclick="agregarProducto()">
    </div>

    <!-- Totales -->
    <div class="total">
        <span>Total sin IGV: <span id="totalSinIVA">S/ 0.00</span></span><br>
        <span>IGV Total: <span id="ivaTotal">S/ 0.00</span></span><br>
        <span>Importe Total: <span id="importeTotal">S/ 0.00</span></span>
    </div>

    <!-- Botón para guardar la factura y ver listado -->
    <input type="button" value="Guardar Factura" onclick="guardarFactura()">

</div>

<!-- Listado de facturas -->
<div class="container listado-section">
    <h2>Listado de Facturas</h2>
    <table class="listado-table">
        <thead>
            <tr>
                <th>Tipo</th>
                <th>Serie</th>
                <th>Numeración</th>
                <th>Fecha</th>
                <th>Importe Total</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody id="facturasList"></tbody>
    </table>
</div>

<script>
    // Lista de facturas
    let facturas = [];

    // Función para actualizar el tipo de moneda
    let IGV = 18;
    let moneda = 'PEN';
    let simboloMoneda = 'S/';

    function actualizarMoneda() {
        moneda = document.getElementById('moneda').value;
        simboloMoneda = moneda === 'PEN' ? 'S/' : '$';
        calcularTotales();
    }

    // Función para agregar productos
    function agregarProducto() {
        const table = document.getElementById('productosTable').getElementsByTagName('tbody')[0];
        const newRow = table.insertRow();

        newRow.innerHTML = `
            <td><input type="text" class="descripcion" placeholder="Descripción del producto"></td>
            <td><input type="number" class="cantidad" min="1" placeholder="Cantidad" onchange="calcularTotales()"></td>
            <td><input type="number" class="precioUnitario" min="0" placeholder="Precio Unitario" onchange="calcularTotales()"></td>
            <td><input type="number" class="baseImponible" min="0" placeholder="Base Imponible" readonly></td>
            <td><input type="number" class="iva" min="0" placeholder="IGV" readonly></td>
            <td><input type="number" class="importeTotal" min="0" placeholder="Importe Total" readonly></td>
            <td><input type="button" class="eliminar" value="Eliminar" onclick="eliminarFila(this)"></td>
        `;
    }

    // Función para eliminar una fila
    function eliminarFila(button) {
        const row = button.parentNode.parentNode;
        row.parentNode.removeChild(row);
        calcularTotales();
    }

    // Función para calcular totales
    function calcularTotales() {
        let totalSinIGV = 0;
        let totalIGV = 0;
        let totalImporte = 0;

        const rows = document.querySelectorAll('#productosTable tbody tr');
        rows.forEach(row => {
            const cantidad = parseFloat(row.querySelector('.cantidad').value) || 0;
            const precioUnitario = parseFloat(row.querySelector('.precioUnitario').value) || 0;
            const baseImponible = cantidad * precioUnitario;
            const igv = baseImponible * (IGV / 100);
            const importeTotal = baseImponible + igv;

            row.querySelector('.baseImponible').value = baseImponible.toFixed(2);
            row.querySelector('.iva').value = igv.toFixed(2);
            row.querySelector('.importeTotal').value = importeTotal.toFixed(2);

            totalSinIGV += baseImponible;
            totalIGV += igv;
            totalImporte += importeTotal;
        });

        // Actualizar los totales
        document.getElementById('totalSinIVA').textContent = `${simboloMoneda} ${totalSinIGV.toFixed(2)}`;
        document.getElementById('ivaTotal').textContent = `${simboloMoneda} ${totalIGV.toFixed(2)}`;
        document.getElementById('importeTotal').textContent = `${simboloMoneda} ${totalImporte.toFixed(2)}`;
    }

    // Función para guardar una factura
    function guardarFactura() {
        const tipoComprobante = document.getElementById('tipoComprobante').value;
        const serie = document.getElementById('serie').value;
        const numeracion = document.getElementById('numeracion').value;
        const fecha = document.getElementById('fecha').value;
        const importeTotal = document.getElementById('importeTotal').textContent;

        // Añadir la factura a la lista
        facturas.push({
            tipoComprobante,
            serie,
            numeracion,
            fecha,
            importeTotal
        });

        // Actualizar el listado de facturas
        actualizarListadoFacturas();
    }

    // Función para actualizar el listado de facturas
    function actualizarListadoFacturas() {
        const tbody = document.getElementById('facturasList');
        tbody.innerHTML = '';
        
        facturas.forEach((factura, index) => {
            const row = document.createElement('tr');
            row.innerHTML = `
                <td>${factura.tipoComprobante}</td>
                <td>${factura.serie}</td>
                <td>${factura.numeracion}</td>
                <td>${factura.fecha}</td>
                <td>${factura.importeTotal}</td>
                <td><button onclick="descargarPDF(${index})">Descargar PDF</button></td>
            `;
            tbody.appendChild(row);
        });
    }

    // Función para descargar la factura en PDF
    function descargarPDF(index) {
        const { jsPDF } = window.jspdf;
        const factura = facturas[index];

        const doc = new jsPDF();

        doc.setFont("helvetica", "normal");
        doc.setFontSize(12);

        doc.text(`Factura ${factura.tipoComprobante}`, 10, 10);
        doc.text(`Serie: ${factura.serie} - Numeración: ${factura.numeracion}`, 10, 20);
        doc.text(`Fecha: ${factura.fecha}`, 10, 30);
        doc.text(`Importe Total: ${factura.importeTotal}`, 10, 40);

        doc.save(`Factura_${factura.serie}_${factura.numeracion}.pdf`);
    }
</script>

</body>
</html>
