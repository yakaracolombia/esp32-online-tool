volver al [INICIO ](index.md).

<img src="imagenes/rp2040_mod.png" height="250">

###  **GRABAR FIRMWARE RP2040**

<div id="flash-container" style="background: #1e1e1e; padding: 20px; border-radius: 10px; text-align: center; color: white; border: 1px solid #333;">
    <button id="btn-conectar" style="padding: 12px 25px; font-size: 16px; background-color: #e84a5f; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">
        CONECTAR RP2040
    </button>
    <div id="status" style="margin-top: 15px; font-family: monospace; color: #aaa;">Estado: Esperando clic...</div>
</div>

<script>
(function() {
    const btn = document.getElementById('btn-conectar');
    const status = document.getElementById('status');

    btn.onclick = async () => {
        // 1. Diagnóstico inmediato
        if (!window.isSecureContext) {
            alert("ERROR: WebUSB solo funciona en sitios con HTTPS o en localhost.");
            status.textContent = "Error: Sitio no seguro (requiere HTTPS)";
            return;
        }

        if (!navigator.usb) {
            alert("ERROR: Tu navegador no soporta o tiene bloqueado WebUSB.");
            status.textContent = "Error: Navegador no compatible";
            return;
        }

        try {
            status.textContent = "Estado: Abriendo selector de dispositivos...";
            
            // 2. Abrir el selector (Esto DEBE abrir la ventana del navegador)
            const device = await navigator.usb.requestDevice({
                filters: [{ vendorId: 0x2e8a }] // ID de Raspberry Pi
            });

            status.textContent = "Conectado a: " + device.productName;
            status.style.color = "#00ff00";

            // Aquí iría la lógica de envío del archivo .uf2
            // Por ahora, si llegas aquí, ¡ya lograste que el navegador vea el RP2040!
            alert("¡Dispositivo detectado! Conectado a " + device.productName);

        } catch (err) {
            console.error(err);
            if (err.name === "NotFoundError") {
                status.textContent = "Estado: No seleccionaste ningún dispositivo.";
            } else {
                status.textContent = "Error: " + err.message;
            }
            status.style.color = "#ff4444";
        }
    };
})();
</script>

<img src="imagenes/line.png" height="5">