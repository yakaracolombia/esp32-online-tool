volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

#### ¿Programación de RP2040 vía Web?
Así como lo hacemos con el ESP32, ahora podemos aprovechar la tecnología **WebUSB** para grabar archivos `.uf2` directamente en nuestra Raspberry Pi Pico o cualquier placa basada en RP2040 sin necesidad de arrastrar archivos a carpetas.

he creado esta sección para simplificar el proceso de instalación en nuevos proyectos basados en este microcontrolador.

<img src="imagenes/line.png" height="5">

### Instrucciones de Instalación:
1. Pon tu RP2040 en modo **BOOTSEL** (conéctalo al PC mientras mantienes presionado el botón BOOTSEL).
2. Haz clic en el botón **"CONECTAR Y GRABAR"** de abajo.
3. Selecciona el dispositivo llamado **"RP2 Boot"** en la lista que abrirá el navegador.
4. Espera a que la barra de progreso termine; el dispositivo se reiniciará solo con el nuevo firmware.

<img src="imagenes/line.png" height="5">

###  **GRABAR FIRMWARE DE PRUEBA** (test.uf2)

<!-- Contenedor del Programador -->
<div id="flash-container" style="background: #1e1e1e; padding: 20px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="connect-button" style="padding: 12px 25px; font-size: 16px; background-color: #00a8e8; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">
        CONECTAR Y GRABAR
    </button>
    <div id="status" style="margin-top: 15px; font-family: monospace; color: #aaa;">Estado: Esperando conexión...</div>
    <div id="progress-bar" style="width: 0%; height: 10px; background-color: #00ff00; margin-top: 10px; border-radius: 5px; transition: width 0.3s;"></div>
</div>

<!-- Script del Programador basado en Adafruit WebUSB Services -->
<script type="module">
  import { UF2 } from "https://adafruit.github.io/Adafruit_WebUSB_Services/uf2.js";

  const connectButton = document.getElementById('connect-button');
  const status = document.getElementById('status');
  const progressBar = document.getElementById('progress-bar');

  connectButton.onclick = async () => {
    try {
      status.textContent = "Estado: Buscando dispositivo...";
      
      // 1. Cargar el archivo .uf2 (Asegúrate de que la ruta sea correcta)
      const response = await fetch('firmware/test.uf2');
      if (!response.ok) throw new Error("No se encontró el archivo test.uf2");
      const buffer = await response.arrayBuffer();

      // 2. Conectar vía WebUSB
      const device = await navigator.usb.requestDevice({
        filters: [{ vendorId: 0x2e8a, productId: 0x0003 }] // RP2040 Bootloader
      });

      const uf2 = new UF2(device);
      await uf2.connect();

      status.textContent = "Estado: Grabando...";
      
      // 3. Flashear
      await uf2.flash(buffer, (current, total) => {
        const percent = Math.round((current / total) * 100);
        progressBar.style.width = percent + "%";
        status.textContent = `Estado: Grabando ${percent}%`;
      });

      status.textContent = "Estado: ¡Grabado con éxito!";
      status.style.color = "#00ff00";
      
    } catch (err) {
      console.error(err);
      status.textContent = "Error: " + err.message;
      status.style.color = "#ff4444";
    }
  };
</script>

<img src="imagenes/line.png" height="5">

#### ¿No detecta el dispositivo?
Recuerda que para que el navegador vea el RP2040, este debe estar estrictamente en modo **BOOTSEL**. Si usas Windows, no necesitas drivers adicionales para el modo bootloader, pero asegúrate de usar un cable de datos de buena calidad.

<img src="imagenes/line.png" height="5">