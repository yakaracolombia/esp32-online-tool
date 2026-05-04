volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

###  **GRABADOR DE FIRMWARE RP2040**

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
// Función para cargar scripts de forma dinámica sin romper el botón
function loadScript(src) {
    return new Promise((resolve, reject) => {
        const script = document.createElement('script');
        script.src = src;
        script.type = 'module';
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
    });
}

const btnFlash = document.getElementById('btn-flash');
const statusText = document.getElementById('status-text');
const progressBar = document.getElementById('progress-bar');

btnFlash.onclick = async () => {
    statusText.textContent = "Estado: Iniciando...";
    statusText.style.color = "#aaa";

    try {
        // 1. Abrir el selector (Esto funcionará porque no hay imports previos que lo bloqueen)
        const device = await navigator.usb.requestDevice({
            filters: [{ vendorId: 0x2e8a }] 
        });

        statusText.textContent = "Conectado: " + device.productName;

        // 2. Cargar la librería solo después de que el usuario dio permiso
        statusText.textContent = "Estado: Cargando motor de grabado...";
        const module = await import("https://adafruit.github.io/Adafruit_WebUSB_Services/uf2.js");
        const UF2 = module.UF2;

        // 3. Descargar el firmware
        statusText.textContent = "Estado: Descargando firmware...";
        const response = await fetch('firmware/test.uf2');
        if (!response.ok) throw new Error("No se encontró el archivo firmware/test.uf2");
        const buffer = await response.arrayBuffer();

        // 4. Iniciar grabado
        const uf2 = new UF2(device);
        await uf2.connect();

        statusText.textContent = "Estado: Grabando...";
        await uf2.flash(buffer, (current, total) => {
            const percent = Math.round((current / total) * 100);
            progressBar.style.width = percent + "%";
            statusText.textContent = `Estado: Grabando ${percent}%`;
        });

        statusText.textContent = "¡GRABADO EXITOSO!";
        statusText.style.color = "#00ff00";

    } catch (err) {
        console.error(err);
        statusText.style.color = "#ff4d4d";
        if (err.name === "NotFoundError") {
            statusText.textContent = "Estado: No seleccionaste dispositivo.";
        } else {
            statusText.textContent = "Error: " + err.message;
        }
    }
};
</script>

<img src="imagenes/line.png" height="5">