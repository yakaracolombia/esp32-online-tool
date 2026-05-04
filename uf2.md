async function flashRP2040(device, buffer, progressCallback) {
    await device.open();
    if (device.configuration === null) await device.selectConfiguration(1);
    
    // El RP2040 suele tener 2 interfaces. Buscamos la que maneja WebUSB/Bulk
    // Normalmente es la interfaz 1, pero vamos a asegurar la conexión.
    const interfaceNumber = 1; 
    await device.claimInterface(interfaceNumber);

    // BUSCAR EL ENDPOINT DE SALIDA AUTOMÁTICAMENTE
    // No todos los RP2040 usan el mismo ID de endpoint.
    const alternate = device.configuration.interfaces[interfaceNumber].alternates[0];
    const outEndpoint = alternate.endpoints.find(e => e.direction === 'out');
    
    if (!outEndpoint) {
        throw new Error("No se encontró un canal de salida (Endpoint Out) en el dispositivo.");
    }

    const endpointNumber = outEndpoint.endpointNumber;
    const blocks = new Uint8Array(buffer);
    const blockSize = 512;
    const numBlocks = blocks.length / blockSize;

    for (let i = 0; i < numBlocks; i++) {
        const block = blocks.slice(i * blockSize, (i + 1) * blockSize);
        
        try {
            // Usamos el endpoint detectado automáticamente
            await device.transferOut(endpointNumber, block);
        } catch (e) {
            throw new Error(`Error en bloque ${i}: ${e.message}`);
        }
        
        if (progressCallback) progressCallback(i + 1, numBlocks);
    }

    // El RP2040 se reinicia automáticamente al recibir el último bloque UF2 válido
    try {
        await device.releaseInterface(interfaceNumber);
        await device.close();
    } catch (e) {
        // Es normal que falle aquí porque el dispositivo ya se reinició
        console.log("El dispositivo se reinició correctamente.");
    }
}