<!DOCTYPE html><html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Registro de Notas Trimestrales</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
    }
    table {
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    td, th {
      border: 1px solid #ccc;
      padding: 6px;
      text-align: center;
    }
    input[type="number"], input[type="text"] {
      width: 60px;
      text-align: center;
    }
    .section {
      margin-bottom: 20px;
    }
    .resultado {
      font-weight: bold;
      font-size: 1.2em;
    }
    .titulo-trimestre {
      font-size: 1.1em;
      font-weight: bold;
      margin-top: 30px;
    }
    @media print {
      button {
        display: none;
      }
      input {
        border: none;
      }
    }
  </style>
</head>
<body>
  <h1>Registro de Notas Trimestrales</h1>  <div class="section">
    <label>Nombre del Estudiante: <input type="text" id="nombre-estudiante"></label><br><br>
    <label>Curso: <input type="text" id="curso-estudiante"></label>
  </div>  <!-- Sección para los tres trimestres -->  <div id="trimestres">
    <script>
      for (let t = 1; t <= 3; t++) {
        document.write(`
        <div class="section">
          <div class="titulo-trimestre">Trimestre ${t}</div>
          <table>
            <tr>
              <th colspan="10">Tareas Individuales</th>
              <th>Promedio</th>
            </tr>
            <tr>
              ${[...Array(10)].map((_, i) => `<td><input type="number" min="0" max="10" class="t-ind-${t}" step="0.01"></td>`).join('')}
              <td id="prom-tind-${t}">0.00</td>
            </tr>
          </table><table>
        <tr>
          <th colspan="10">Tareas Grupales</th>
          <th>Promedio</th>
        </tr>
        <tr>
          ${[...Array(10)].map((_, i) => `<td><input type="number" min="0" max="10" class="t-grup-${t}" step="0.01"></td>`).join('')}
          <td id="prom-tgrup-${t}">0.00</td>
        </tr>
      </table>

      <table>
        <tr>
          <th>Proyecto Interdisciplinario</th>
          <th>Examen Trimestral</th>
          <th>Promedio Trimestral</th>
        </tr>
        <tr>
          <td><input type="number" min="0" max="10" id="proyecto-${t}" step="0.01"></td>
          <td><input type="number" min="0" max="10" id="examen-${t}" step="0.01"></td>
          <td id="prom-final-${t}">0.00</td>
        </tr>
      </table>
    </div>
    `);
  }
</script>

  </div>  <div class="section">
    <button onclick="calcularTodo()">Calcular Notas Finales</button>
    <button onclick="window.print()">Imprimir Reporte</button>
    <button onclick="generarPDF()">Descargar PDF</button>
  </div>  <div class="section">
    <h2>Promedio Anual</h2>
    <p class="resultado">Nombre: <span id="nombre-mostrar">-</span></p>
    <p class="resultado">Curso: <span id="curso-mostrar">-</span></p>
    <p class="resultado">Promedio Anual: <span id="prom-anual">0</span></p>
    <p class="resultado">Estado del Estudiante: <span id="estado-estudiante">-</span></p>
  </div>  <script>
    function calcularPromedioClase(selector) {
      const inputs = document.querySelectorAll(selector);
      let suma = 0, conteo = 0;
      inputs.forEach(input => {
        const val = parseFloat(input.value);
        if (!isNaN(val)) {
          suma += val;
          conteo++;
        }
      });
      return conteo ? (suma / conteo) : 0;
    }

    function calcularTrimestre(num) {
      const promTInd = calcularPromedioClase(`.t-ind-${num}`);
      const promTGrup = calcularPromedioClase(`.t-grup-${num}`);
      const promActividades = (promTInd + promTGrup) / 2;
      const proyecto = parseFloat(document.getElementById(`proyecto-${num}`).value) || 0;
      const examen = parseFloat(document.getElementById(`examen-${num}`).value) || 0;
      const promFinal = (promActividades * 0.7 + ((proyecto + examen) / 2) * 0.3).toFixed(2);

      document.getElementById(`prom-tind-${num}`).textContent = promTInd.toFixed(2);
      document.getElementById(`prom-tgrup-${num}`).textContent = promTGrup.toFixed(2);
      document.getElementById(`prom-final-${num}`).textContent = promFinal;

      return parseFloat(promFinal);
    }

    function calcularTodo() {
      const p1 = calcularTrimestre(1);
      const p2 = calcularTrimestre(2);
      const p3 = calcularTrimestre(3);
      const promAnual = ((p1 + p2 + p3) / 3).toFixed(2);

      document.getElementById("prom-anual").textContent = promAnual;

      const estado = promAnual < 7.00 ? "Supletorio" : "Aprueba el Año";
      document.getElementById("estado-estudiante").textContent = estado;

      document.getElementById("nombre-mostrar").textContent = document.getElementById("nombre-estudiante").value || "-";
      document.getElementById("curso-mostrar").textContent = document.getElementById("curso-estudiante").value || "-";
    }

    async function generarPDF() {
      const {{ jsPDF }} = window.jspdf;
      const doc = new jsPDF();
      const nombre = document.getElementById("nombre-estudiante").value || "-";
      const curso = document.getElementById("curso-estudiante").value || "-";
      const promedio = document.getElementById("prom-anual").textContent;
      const estado = document.getElementById("estado-estudiante").textContent;

      doc.setFontSize(14);
      doc.text("REPORTE DE NOTAS TRIMESTRALES", 20, 20);
      doc.text(`Nombre: ${nombre}`, 20, 40);
      doc.text(`Curso: ${curso}`, 20, 50);
      doc.text(`Promedio Anual: ${promedio}`, 20, 60);
      doc.text(`Estado: ${estado}`, 20, 70);

      doc.save(`reporte_${nombre.replace(/\s+/g, '_')}.pdf`);
    }
  </script></body>
</html>
