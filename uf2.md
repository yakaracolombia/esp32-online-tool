volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

#### Programación Web para RP2040
Esta sección permite grabar el firmware en dispositivos basados en el chip **RP2040** (como Raspberry Pi Pico) directamente desde el navegador, sin necesidad de arrastrar archivos manualmente a la unidad de disco.

<img src="imagenes/line.png" height="5">

### Instrucciones:
1. Conecta tu RP2040 manteniendo presionado el botón **BOOTSEL**.
2. Asegúrate de que el PC lo reconozca como una unidad de disco llamada `RPI-RP2`.
3. Haz clic en el botón de abajo y selecciona **"RP2 Boot"**.
4. No cierres la pestaña ni desconectes el cable hasta que el estado indique "EXITOSO".

<img src="imagenes/line.png" height="5">

###  **GRABADOR DE FIRMWARE RP2040**

<div id="flash-container" style="background: #1e1e1e; padding: 25px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="btn-flash" style="padding: 15px 30px; font-size: 18px; background-color: #00cecb; color: #1e1e1e; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; transition: 0.3s;">
        CONECTAR Y GRABAR TEST.UF2
    </button>
    
    <div id="status-box" style="margin-top: 20px;">
        <div id="status-text" style="font-family: monospace; color: #aaa; margin-bottom: 10px;">Estado: Esperando...</div>
        <div style="width: 100%; background: #333; height: 12px; border-radius: 6px; overflow: hidden;">
            <div id="progress-bar" style="width: 0%; height: 100%; background: #00cecb; transition: width 0.1s;"></div>
        </div>
    </div>
</div>

<script>
/**
 * Motor de grabado UF2 optimizado para evitar errores de transferencia
 */
async function flashRP2040(device, buffer, progressCallback) {
    await device.open();
    if (device.configuration === null) await device.selectConfiguration(1);
    
    // La mayoría de los bootloaders RP2040 usan la interfaz 1 para WebUSB
    const interfaceNumber = 1; 
    await device.claimInterface(interfaceNumber);

    // Detección dinámica de Endpoint para evitar errores de "transferOut"
    const alternate = device.configuration.interfaces[interfaceNumber].alternates[0];
    const outEndpoint = alternate.endpoints.find(e => e.direction === 'out');
    
    if (!outEndpoint) {
        throw new Error("No se detectó un canal de salida válido en el dispositivo.");
    }

    const endpointNumber = outEndpoint.endpointNumber;
    const blocks = new Uint8Array(buffer);
    const blockSize = 512; // Estándar UF2
    const numBlocks = blocks.length / blockSize;

    for (let i = 0; i < numBlocks; i++) {
        const block = blocks.slice(i * blockSize, (i + 1) * blockSize);
        
        try {
            await device.transferOut(endpointNumber, block);
        } catch (e) {
            throw new Error(`Fallo en el bloque ${i}: ${e.message}`);
        }
        
        if (progressCallback) progressCallback(i + 1, numBlocks);
    }

    // El dispositivo suele reiniciarse solo tras el último bloque UF2.
    // Intentamos cerrar la conexión de forma limpia.
    try {
        await device.releaseInterface(interfaceNumber);
        await device.close();
    } catch (e) {
        console.log("Reinicio detectado.");
    }
}

const btnFlash = document.getElementById('btn-flash');
const statusText = document.getElementById('status-text');
const progressBar = document.getElementById('progress-bar');

btnFlash.onclick = async () => {
    statusText.style.color = "#aaa";
    progressBar.style.width = "0%";

    // Verificación de seguridad
    if (!window.isSecureContext) {
        alert("WebUSB requiere una conexión segura (HTTPS).");
        return;
    }

    try {
        // 1. Solicitar dispositivo
        const device = await navigator.usb.requestDevice({
            filters: [{ vendorId: 0x2e8a, productId: 0x0003 }] 
        });

        statusText.textContent = "Estado: Descargando firmware...";
        
        // 2. Cargar el archivo .uf2 (Asegúrate de que la ruta sea correcta)
        const response = await fetch('firmware/test.uf2');
        if (!response.ok) throw new Error("No se encontró 'firmware/test.uf2' en el servidor.");
        const buffer = await response.arrayBuffer();

        statusText.textContent = "Estado: Iniciando grabado...";
        statusText.style.color = "#00cecb";

        // 3. Ejecutar flasheo
        await flashRP2040(device, buffer, (current, total) => {
            const percent = Math.round((current / total) * 100);
            progressBar.style.width = percent + "%";
            statusText.textContent = `Estado: Grabando ${percent}%`;
        });

        statusText.textContent = "¡GRABADO EXITOSO!";
        statusText.style.color = "#00ff00";
        progressBar.style.background = "#00ff00";
        alert("El firmware se ha grabado correctamente.");

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

#### ¿Problemas comunes?
Si obtienes un error de transferencia, intenta lo siguiente:
*   Usa un puerto USB directo del PC (evita HUBs USB).
*   Cierra cualquier ventana del explorador de archivos que tenga abierta la unidad del RP2040.
*   Asegúrate de que el cable USB sea de buena calidad y de datos.

<img src="imagenes/line.png" height="5">