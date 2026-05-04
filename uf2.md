volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

###  **GRABADOR DE FIRMWARE RP2040 (FIX TRANSFER)**

<div id="flash-container" style="background: #1e1e1e; padding: 25px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="btn-flash" style="padding: 15px 30px; font-size: 18px; background-color: #f39c12; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">
        GRABAR FIRMWARE AHORA
    </button>
    
    <div id="status-box" style="margin-top: 20px;">
        <div id="status-text" style="font-family: monospace; color: #aaa; margin-bottom: 10px;">Estado: Listo para conectar</div>
        <div style="width: 100%; background: #333; height: 12px; border-radius: 6px; overflow: hidden;">
            <div id="progress-bar" style="width: 0%; height: 100%; background: #f39c12; transition: width 0.1s;"></div>
        </div>
    </div>
</div>

<script>
/**
 * Motor de flasheo con Reset de Interfaz para evitar 'Transfer Error'
 */
async function flashRP2040(device, buffer, progressCallback) {
    await device.open();
    
    // 1. Resetear el dispositivo para limpiar errores previos
    await device.reset();
    
    if (device.configuration === null) await device.selectConfiguration(1);
    
    // 2. Buscar la interfaz correcta (Clase 255 es Vendor Specific, típica del Bootloader)
    const interfaceNumber = device.configuration.interfaces.findIndex(iface => 
        iface.alternates[0].interfaceClass === 255
    ) || 1;

    try {
        await device.claimInterface(interfaceNumber);
    } catch (e) {
        throw new Error("El sistema operativo bloquea el acceso. Cierra la carpeta del RP2040.");
    }

    const alternate = device.configuration.interfaces[interfaceNumber].alternates[0];
    const outEndpoint = alternate.endpoints.find(e => e.direction === 'out');
    const endpointNumber = outEndpoint ? outEndpoint.endpointNumber : 2; // Default a 2 si falla detección

    const blocks = new Uint8Array(buffer);
    const numBlocks = blocks.length / 512;

    for (let i = 0; i < numBlocks; i++) {
        const block = blocks.slice(i * 512, (i + 1) * 512);
        
        // Enviamos el bloque
        await device.transferOut(endpointNumber, block);
        
        if (progressCallback) progressCallback(i + 1, numBlocks);
    }

    // El RP2040 se reinicia solo al terminar un UF2 válido
    try { await device.close(); } catch (e) {}
}

const btnFlash = document.getElementById('btn-flash');
const statusText = document.getElementById('status-text');
const progressBar = document.getElementById('progress-bar');

btnFlash.onclick = async () => {
    try {
        const device = await navigator.usb.requestDevice({
            filters: [{ vendorId: 0x2e8a, productId: 0x0003 }] 
        });

        statusText.textContent = "Estado: Cargando archivo...";
        const response = await fetch('firmware/test.uf2');
        if (!response.ok) throw new Error("Archivo firmware/test.uf2 no encontrado.");
        const buffer = await response.arrayBuffer();

        statusText.textContent = "Estado: Grabando...";
        statusText.style.color = "#f39c12";

        await flashRP2040(device, buffer, (current, total) => {
            const percent = Math.round((current / total) * 100);
            progressBar.style.width = percent + "%";
            statusText.textContent = `Estado: Grabando ${percent}%`;
        });

        statusText.textContent = "¡GRABADO EXITOSO!";
        statusText.style.color = "#00ff00";
        progressBar.style.background = "#00ff00";

    } catch (err) {
        console.error(err);
        statusText.style.color = "#ff4d4d";
        statusText.textContent = "Error: " + err.message;
    }
};
</script>

<img src="imagenes/line.png" height="5">