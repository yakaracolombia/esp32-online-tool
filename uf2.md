volver al [INICIO ](index.md).


<img src="imagenes/line.png"
height="5">
###  **"BLUERETRO"** para ESP32 

<script type="module" src="install-button.js?module"></script>
<esp-web-install-button manifest="firmware/firmware_build/blueretro/ogx/manifest.json"></esp-web-install-button>

<img src="imagenes/line.png"
de
height="5">
<img src="imagenes/rp2040_mod.png" height="250">

###  **GRABADOR DE FIRMWARE OGX-MINI RP2040 / CON ESP32**

<div id="flash-container" style="background: #1e1e1e; padding: 25px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="btn-flash" style="padding: 15px 30px; font-size: 18px; background-color: #00cecb; color: #1e1e1e; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; transition: 0.3s;">
        GRABAR TEST.UF2
    </button>
    
    <div id="status-box" style="margin-top: 20px;">
        <div id="status-text" style="font-family: monospace; color: #aaa; margin-bottom: 10px;">Estado: Esperando conexión...</div>
        <div style="width: 100%; background: #333; height: 12px; border-radius: 6px; overflow: hidden;">
            <div id="progress-bar" style="width: 0%; height: 100%; background: #00cecb; transition: width 0.2s;"></div>
        </div>
    </div>
</div>

<script type="module">
// Importamos la librería de servicios UF2 directamente de los servidores de Adafruit
import { UF2 } from "https://adafruit.github.io/Adafruit_WebUSB_Services/uf2.js";

const btnFlash = document.getElementById('btn-flash');
const statusText = document.getElementById('status-text');
const progressBar = document.getElementById('progress-bar');

btnFlash.onclick = async () => {
    try {
        // 1. Limpiar estado
        progressBar.style.width = "0%";
        statusText.style.color = "#aaa";
        
        // 2. Cargar el archivo .uf2 desde tu servidor
        statusText.textContent = "Estado: Descargando firmware...";
        const response = await fetch('firmware/test.uf2'); // Verifica que esta ruta sea correcta
        if (!response.ok) throw new Error("No se pudo descargar el archivo .uf2");
        const buffer = await response.arrayBuffer();

        // 3. Solicitar conexión al RP2040
        statusText.textContent = "Estado: Selecciona tu RP2040...";
        const device = await navigator.usb.requestDevice({
            filters: [{ vendorId: 0x2e8a, productId: 0x0003 }] // Modo Bootloader RP2
        });

        // 4. Iniciar el motor UF2
        const uf2 = new UF2(device);
        await uf2.connect();

        statusText.textContent = "Estado: Grabando firmware...";
        statusText.style.color = "#00cecb";

        // 5. Proceso de flasheo con barra de progreso
        await uf2.flash(buffer, (current, total) => {
            const percent = Math.round((current / total) * 100);
            progressBar.style.width = percent + "%";
            statusText.textContent = `Estado: Grabando... ${percent}%`;
        });

        // 6. Finalización
        statusText.textContent = "Estado: ¡GRABADO EXITOSO! Reiniciando...";
        statusText.style.color = "#00ff00";
        progressBar.style.background = "#00ff00";

        alert("Firmware grabado correctamente. El dispositivo se reiniciará solo.");

    } catch (err) {
        console.error(err);
        statusText.style.color = "#ff4d4d";
        if (err.name === "NotFoundError") {
            statusText.textContent = "Estado: Cancelado por el usuario.";
        } else {
            statusText.textContent = "Error: " + err.message;
        }
    }
};
</script>

<img src="imagenes/line.png" height="5">