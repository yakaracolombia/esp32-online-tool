volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

###  **GRABADOR DE FIRMWARE RP2040 (SOPORTE OFFLINE)**

<div id="flash-container" style="background: #1e1e1e; padding: 25px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="btn-flash" style="padding: 15px 30px; font-size: 18px; background-color: #00cecb; color: #1e1e1e; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">
        CONECTAR Y GRABAR RP2040
    </button>
    
    <div id="status-box" style="margin-top: 20px;">
        <div id="status-text" style="font-family: monospace; color: #aaa; margin-bottom: 10px;">Estado: Esperando...</div>
        <div style="width: 100%; background: #333; height: 12px; border-radius: 6px; overflow: hidden;">
            <div id="progress-bar" style="width: 0%; height: 100%; background: #00cecb; transition: width 0.1s;"></div>
        </div>
    </div>
</div>

<script>
// Motor UF2 integrado (Versión minimalista para evitar dependencias externas)
const UF2_MAGIC_START0 = 0x0A324655; // "UF2\n"
const UF2_MAGIC_START1 = 0x9E5D5157;
const UF2_MAGIC_END = 0x0AB16F30;

async function flashRP2040(device, buffer, progressCallback) {
    await device.open();
    if (device.configuration === null) await device.selectConfiguration(1);
    
    // El RP2040 Bootloader usualmente usa la interfaz 1 para WebUSB
    await device.claimInterface(1);

    const blocks = new Uint8Array(buffer);
    const numBlocks = blocks.length / 512;

    for (let i = 0; i < numBlocks; i++) {
        const block = blocks.slice(i * 512, (i + 1) * 512);
        
        // Enviamos el bloque al endpoint de salida (usualmente 3 en RP2040)
        await device.transferOut(3, block);
        
        if (progressCallback) progressCallback(i + 1, numBlocks);
    }

    await device.releaseInterface(1);
    await device.close();
}

const btnFlash = document.getElementById('btn-flash');
const statusText = document.getElementById('status-text');
const progressBar = document.getElementById('progress-bar');

btnFlash.onclick = async () => {
    statusText.style.color = "#aaa";
    progressBar.style.width = "0%";

    try {
        // 1. Selector de dispositivo (Funciona porque no hay imports)
        const device = await navigator.usb.requestDevice({
            filters: [{ vendorId: 0x2e8a, productId: 0x0003 }] 
        });

        statusText.textContent = "Estado: Descargando firmware...";
        
        // 2. Cargar el archivo .uf2
        const response = await fetch('firmware/test.uf2');
        if (!response.ok) throw new Error("No se encontró test.uf2 en /firmware/");
        const buffer = await response.arrayBuffer();

        statusText.textContent = "Estado: Grabando...";
        statusText.style.color = "#00cecb";

        // 3. Flashear usando el motor interno
        await flashRP2040(device, buffer, (current, total) => {
            const percent = Math.round((current / total) * 100);
            progressBar.style.width = percent + "%";
            statusText.textContent = `Estado: Grabando ${percent}%`;
        });

        statusText.textContent = "¡GRABADO EXITOSO!";
        statusText.style.color = "#00ff00";
        alert("Proceso terminado con éxito.");

    } catch (err) {
        console.error(err);
        statusText.style.color = "#ff4d4d";
        statusText.textContent = "Error: " + err.message;
    }
};
</script>

<img src="imagenes/line.png" height="5">