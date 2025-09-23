<script>
  let mensaje = '';
  let clave = '';
  let resultado = '';
  let modo = 'cifrar';
  let error = '';
  let mostrarInfo = false;
  let animacionActiva = false;
  let mensajeCifradoHex = '';

  // Función para validar ASCII
  function validarAscii(texto) {
    for (let i = 0; i < texto.length; i++) {
      const code = texto.charCodeAt(i);
      if (code < 32 || code > 126) {
        return false;
      }
    }
    return true;
  }

  // Validar hexadecimal
  function validarHex(texto) {
    return /^[0-9A-Fa-f]*$/.test(texto) && texto.length % 2 === 0;
  }

  // Generar clave aleatoria
  function generarClave() {
    if (modo === 'cifrar') {
      if (!validarAscii(mensaje) || mensaje.length === 0) {
        error = 'Escribe un mensaje válido en ASCII antes de generar la clave.';
        return;
      }
    } else {
      if (!validarHex(mensaje) || mensaje.length === 0) {
        error = 'Ingresa un texto cifrado válido en hexadecimal antes de generar la clave.';
        return;
      }
    }
    
    animacionActiva = true;
    
    setTimeout(() => {
      let nuevaClave = '';
      const longitud = modo === 'cifrar' ? mensaje.length : mensaje.length / 2;
      
      for (let i = 0; i < longitud; i++) {
        const randCode = Math.floor(Math.random() * (126 - 32 + 1)) + 32;
        nuevaClave += String.fromCharCode(randCode);
      }
      clave = nuevaClave;
      error = '';
      resultado = `🔑 Clave generada: ${nuevaClave}`;
      animacionActiva = false;
    }, 800);
  }

  // Función XOR para cifrar
  function xorCifrar(mensaje, clave) {
    if (mensaje.length !== clave.length) {
      throw new Error('El mensaje y la clave deben tener la misma longitud.');
    }
    let resultado = [];
    for (let i = 0; i < mensaje.length; i++) {
      const m = mensaje.charCodeAt(i);
      const k = clave.charCodeAt(i);
      const xor = m ^ k;
      resultado.push(xor.toString(16).padStart(2, '0'));
    }
    return resultado.join('');
  }

  // Función XOR para descifrar
  function xorDescifrar(cifradoHex, clave) {
    if (cifradoHex.length % 2 !== 0) {
      throw new Error('El texto cifrado en hex no es válido.');
    }
    
    const bytes = [];
    for (let i = 0; i < cifradoHex.length; i += 2) {
      bytes.push(parseInt(cifradoHex.substr(i, 2), 16));
    }
    
    if (bytes.length !== clave.length) {
      throw new Error('El cifrado y la clave deben tener la misma longitud.');
    }
    
    let resultado = '';
    for (let i = 0; i < bytes.length; i++) {
      resultado += String.fromCharCode(bytes[i] ^ clave.charCodeAt(i));
    }
    return resultado;
  }

  // Cifrar mensaje
  function cifrar() {
    if (!validarAscii(mensaje) || !validarAscii(clave)) {
      error = 'Solo se permiten caracteres ASCII imprimibles.';
      return;
    }
    
    if (mensaje.length !== clave.length) {
      error = 'El mensaje y la clave deben tener la misma longitud.';
      return;
    }
    
    animacionActiva = true;
    
    setTimeout(() => {
      try {
        const cifrado = xorCifrar(mensaje, clave);
        mensajeCifradoHex = cifrado;
        resultado = cifrado;
        error = '';
      } catch (e) {
        error = 'Error: ' + e.message;
      }
      animacionActiva = false;
    }, 600);
  }

  // Descifrar mensaje
  function descifrar() {
    if (!validarHex(mensaje)) {
      error = 'El texto cifrado debe estar en formato hexadecimal válido.';
      return;
    }
    
    if (!validarAscii(clave)) {
      error = 'La clave debe contener solo caracteres ASCII imprimibles.';
      return;
    }
    
    animacionActiva = true;
    
    setTimeout(() => {
      try {
        const resultadoDescifrado = xorDescifrar(mensaje, clave);
        resultado = resultadoDescifrado;
        error = '';
      } catch (e) {
        error = 'Error: ' + e.message;
      }
      animacionActiva = false;
    }, 600);
  }

  // Cambiar modo
  function toggleModo(nuevoModo) {
    modo = nuevoModo;
    error = '';
    resultado = '';
    mensaje = '';
    clave = '';
    mensajeCifradoHex = '';
  }

  // Limpiar campos
  function limpiarCampos() {
    mensaje = '';
    clave = '';
    resultado = '';
    error = '';
    mensajeCifradoHex = '';
  }

  // Copiar resultado al portapapeles
  function copiarResultado() {
    navigator.clipboard.writeText(resultado).then(() => {
      // Mostrar feedback visual temporal
      const originalResultado = resultado;
      resultado = '📋 ¡Copiado al portapapeles!';
      setTimeout(() => {
        resultado = originalResultado;
      }, 2000);
    });
  }

  // Usar último cifrado para descifrar
  function usarUltimoCifrado() {
    if (mensajeCifradoHex) {
      mensaje = mensajeCifradoHex;
      modo = 'descifrar';
    }
  }
</script>

<main>
  <div class="min-h-screen bg-black p-4">
    <!-- Header con animación -->
    <div class="text-center mb-8 animate-fade-in">
      <h1 class="text-4xl font-bold mb-2 bg-gradient-to-r from-green-400 to-cyan-400 bg-clip-text text-transparent">
        🔐 Cifrado Vernam
      </h1>
      <p class="text-green-300 text-lg">
        Implementación interactiva del cifrado perfectamente seguro
      </p>
    </div>

    <!-- Selector de modo con animación -->
    <div class="max-w-4xl mx-auto mb-6">
      <div class="flex justify-center space-x-4 mb-6">
        <button
          on:click={() => toggleModo('cifrar')}
          class="px-6 py-3 rounded-full font-semibold transition-all duration-300 transform hover:scale-105 {modo === 'cifrar'
            ? 'bg-gradient-to-r from-green-500 to-lime-400 text-black shadow-lg shadow-green-500/25'
            : 'bg-gray-900/70 text-green-400 border-2 border-green-400 hover:bg-green-400/10'}"
        >
          🔒 Cifrar
        </button>
        <button
          on:click={() => toggleModo('descifrar')}
          class="px-6 py-3 rounded-full font-semibold transition-all duration-300 transform hover:scale-105 {modo === 'descifrar'
            ? 'bg-gradient-to-r from-cyan-500 to-blue-400 text-black shadow-lg shadow-cyan-500/25'
            : 'bg-gray-900/70 text-cyan-400 border-2 border-cyan-400 hover:bg-cyan-400/10'}"
        >
          🔓 Descifrar
        </button>
      </div>
    </div>

    <div class="max-w-6xl mx-auto grid lg:grid-cols-3 gap-6">
      <!-- Panel principal -->
      <div class="lg:col-span-2 bg-gray-900/90 backdrop-blur-sm rounded-2xl shadow-xl shadow-green-500/10 p-6 border border-green-400/20">
        <div class="space-y-6">
          <!-- Campo de entrada principal -->
          <div class="transform transition-all duration-300 hover:scale-[1.01]">
            <label class="block text-sm font-semibold text-green-400 mb-2">
              {#if modo === 'cifrar'}
                📝 Mensaje a cifrar:
              {:else}
                🔢 Texto cifrado (Hexadecimal):
              {/if}
            </label>
            <textarea
              bind:value={mensaje}
              rows="3"
              class="w-full p-4 bg-gray-800/80 text-green-300 border-2 border-gray-700 rounded-xl focus:border-green-500 focus:ring-2 focus:ring-green-500/30 transition-all duration-300 {modo === 'cifrar' ? 'font-sans' : 'font-mono'} resize-none placeholder-gray-500"
              placeholder={modo === 'cifrar' ? 'Escribe tu mensaje en texto plano...' : 'Pega el texto cifrado en hexadecimal (ej: 0a1b2c)...'}
            ></textarea>
            <div class="text-sm text-gray-400 mt-1 flex justify-between">
              <span>
                {#if modo === 'cifrar'}
                  Caracteres: {mensaje.length}
                {:else}
                  Hex: {mensaje.length} chars ({mensaje.length / 2} bytes)
                {/if}
              </span>
              {#if mensaje.length > 0}
                <span class="transition-colors {(modo === 'cifrar' && validarAscii(mensaje)) || (modo === 'descifrar' && validarHex(mensaje)) ? 'text-green-400' : 'text-red-400'}">
                  {#if modo === 'cifrar'}
                    {validarAscii(mensaje) ? '✓ ASCII válido' : '✗ Solo ASCII (32-126)'}
                  {:else}
                    {validarHex(mensaje) ? '✓ Hex válido' : '✗ Solo hexadecimal'}
                  {/if}
                </span>
              {/if}
            </div>
          </div>

          <!-- Campo de clave -->
          <div class="transform transition-all duration-300 hover:scale-[1.01]">
            <label class="block text-sm font-semibold text-cyan-400 mb-2">
              🗝️ Clave secreta:
            </label>
            <div class="relative">
              <input
                type="text"
                bind:value={clave}
                class="w-full p-4 pr-12 bg-gray-800/80 text-cyan-300 border-2 border-gray-700 rounded-xl focus:border-cyan-500 focus:ring-2 focus:ring-cyan-500/30 transition-all duration-300 font-mono placeholder-gray-500"
                placeholder="Escribe la clave o genera una aleatoria..."
              />
              {#if clave.length > 0 && mensaje.length > 0}
                <div class="absolute right-4 top-1/2 transform -translate-y-1/2">
                  {#if (modo === 'cifrar' && clave.length === mensaje.length) || (modo === 'descifrar' && clave.length === mensaje.length / 2)}
                    <span class="text-green-400 text-xl">✓</span>
                  {:else}
                    <span class="text-red-400 text-xl">✗</span>
                  {/if}
                </div>
              {/if}
            </div>
            <div class="text-sm text-gray-400 mt-1 flex justify-between">
              <span>Caracteres: {clave.length}</span>
              <span class="transition-colors {
                (modo === 'cifrar' && clave.length === mensaje.length && mensaje.length > 0) ||
                (modo === 'descifrar' && clave.length === mensaje.length / 2 && mensaje.length > 0)
                ? 'text-green-400' 
                : 'text-yellow-400'}">
                {#if mensaje.length > 0}
                  Necesita: {modo === 'cifrar' ? mensaje.length : mensaje.length / 2}
                {:else}
                  {modo === 'cifrar' ? 'Misma longitud que el mensaje' : 'Longitud = bytes del hex'}
                {/if}
              </span>
            </div>
          </div>

          <!-- Botones de acción -->
          <div class="flex flex-wrap gap-3">
            <button
              on:click={generarClave}
              disabled={animacionActiva || mensaje.length === 0}
              class="flex-1 min-w-[140px] px-4 py-3 bg-gradient-to-r from-yellow-500 to-orange-500 text-black rounded-xl font-semibold hover:from-yellow-400 hover:to-orange-400 transform transition-all duration-300 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100 shadow-lg shadow-yellow-500/20"
            >
              {#if animacionActiva}
                <span class="flex items-center justify-center">
                  <div class="animate-spin rounded-full h-4 w-4 border-2 border-black border-t-transparent mr-2"></div>
                  Generando...
                </span>
              {:else}
                🎲 Generar Clave
              {/if}
            </button>
            
            <button
              on:click={modo === 'cifrar' ? cifrar : descifrar}
              disabled={animacionActiva || mensaje.length === 0 || clave.length === 0 || 
                       (modo === 'cifrar' && mensaje.length !== clave.length) ||
                       (modo === 'descifrar' && clave.length !== mensaje.length / 2)}
              class="flex-1 min-w-[120px] px-4 py-3 rounded-xl font-semibold transform transition-all duration-300 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100 shadow-lg {modo === 'cifrar'
                ? 'bg-gradient-to-r from-green-500 to-lime-400 text-black hover:from-green-400 hover:to-lime-300 shadow-green-500/20'
                : 'bg-gradient-to-r from-cyan-500 to-blue-400 text-black hover:from-cyan-400 hover:to-blue-300 shadow-cyan-500/20'}"
            >
              {#if animacionActiva}
                <span class="flex items-center justify-center">
                  <div class="animate-spin rounded-full h-4 w-4 border-2 border-black border-t-transparent mr-2"></div>
                  Procesando...
                </span>
              {:else}
                {modo === 'cifrar' ? '🔒 Cifrar' : '🔓 Descifrar'}
              {/if}
            </button>
            
            <button
              on:click={limpiarCampos}
              class="px-4 py-3 bg-red-600 text-white rounded-xl font-semibold hover:bg-red-500 transform transition-all duration-300 hover:scale-105 shadow-lg shadow-red-600/20"
            >
              🗑️ Limpiar
            </button>
          </div>

          <!-- Panel de resultado -->
          <div class="bg-gray-800/60 rounded-xl p-4 min-h-[100px] border border-gray-700">
            <div class="flex justify-between items-center mb-2">
              <h3 class="text-lg font-semibold text-cyan-300">
                📊 Resultado:
              </h3>
              {#if resultado && !error}
                <button
                  on:click={copiarResultado}
                  class="px-3 py-1 bg-blue-600 text-white text-sm rounded-lg hover:bg-blue-500 transition-colors shadow-md shadow-blue-600/20"
                >
                  📋 Copiar
                </button>
              {/if}
            </div>
            
            <!-- Mensaje de error -->
            {#if error}
              <div class="p-4 bg-red-900/50 border-l-4 border-red-400 rounded-lg animate-shake">
                <div class="flex items-center">
                  <span class="text-red-400 text-xl mr-2">⚠️</span>
                  <p class="text-red-300 font-medium">{error}</p>
                </div>
              </div>
            {/if}
            
            <!-- Resultado exitoso -->
            {#if resultado && !error}
              <div class="p-4 bg-gradient-to-r from-gray-800/80 to-gray-700/80 border-l-4 border-green-400 rounded-lg animate-slide-up">
                <div class="space-y-2">
                  <div class="flex items-center justify-between">
                    <span class="text-sm font-semibold text-green-400">
                      {modo === 'cifrar' ? '🔒 Texto Cifrado (Hex):' : '🔓 Texto Descifrado:'}
                    </span>
                    {#if modo === 'cifrar'}
                      <button
                        on:click={usarUltimoCifrado}
                        class="px-2 py-1 bg-purple-600 text-white text-xs rounded hover:bg-purple-500 transition-colors"
                      >
                        ↻ Usar para descifrar
                      </button>
                    {/if}
                  </div>
                  <p class="text-green-300 {modo === 'cifrar' ? 'font-mono' : 'font-sans'} text-sm break-all leading-relaxed bg-gray-900/80 p-3 rounded border border-gray-600">
                    {resultado}
                  </p>
                  {#if modo === 'cifrar'}
                    <p class="text-xs text-gray-400">
                      💡 Guarda este texto cifrado y la clave para poder descifrar después
                    </p>
                  {/if}
                </div>
              </div>
            {/if}
            
            <!-- Estado vacío -->
            {#if !resultado && !error}
              <div class="text-center py-8 text-gray-500">
                <div class="text-4xl mb-2">🔍</div>
                <p>{modo === 'cifrar' ? 'El texto cifrado aparecerá aquí' : 'El texto descifrado aparecerá aquí'}</p>
              </div>
            {/if}
          </div>
        </div>
      </div>

      <!-- Panel de información -->
      <div class="space-y-6">
        <!-- Información educativa -->
        <div class="bg-gray-900/90 backdrop-blur-sm rounded-2xl shadow-xl shadow-cyan-500/10 p-6 border border-cyan-400/20">
          <button
            on:click={() => mostrarInfo = !mostrarInfo}
            class="w-full flex items-center justify-between text-lg font-semibold text-cyan-300 mb-4 hover:text-cyan-400 transition-colors"
          >
            <span class="flex items-center">
              📚 ¿Cómo funciona?
            </span>
            <span class="transform transition-transform duration-300 {mostrarInfo ? 'rotate-180' : ''}">
              ⌄
            </span>
          </button>
          
          <div class="space-y-4 transition-all duration-500 overflow-hidden {mostrarInfo ? 'max-h-[800px] opacity-100' : 'max-h-0 opacity-0'}">
            <!-- Proceso visual -->
            <div class="bg-gradient-to-r from-gray-800/80 to-gray-700/80 p-4 rounded-xl border border-green-400/30">
              <h4 class="font-semibold text-green-400 mb-3">🔄 Proceso de Cifrado:</h4>
              <div class="space-y-3">
                <div class="flex flex-wrap justify-center items-center gap-2 text-sm">
                  <div class="bg-blue-600 text-white px-3 py-2 rounded-lg font-mono">H (72)</div>
                  <div class="text-xl text-cyan-400">⊕</div>
                  <div class="bg-purple-600 text-white px-3 py-2 rounded-lg font-mono">O (79)</div>
                  <div class="text-xl text-cyan-400">=</div>
                  <div class="bg-green-600 text-white px-3 py-2 rounded-lg font-mono">7 (0x07)</div>
                </div>
                <div class="flex flex-wrap justify-center items-center gap-2 text-sm">
                  <div class="bg-blue-600 text-white px-3 py-2 rounded-lg font-mono">I (73)</div>
                  <div class="text-xl text-cyan-400">⊕</div>
                  <div class="bg-purple-600 text-white px-3 py-2 rounded-lg font-mono">K (75)</div>
                  <div class="text-xl text-cyan-400">=</div>
                  <div class="bg-green-600 text-white px-3 py-2 rounded-lg font-mono">2 (0x02)</div>
                </div>
                <div class="text-center">
                  <div class="bg-green-600 text-white px-4 py-2 rounded-lg font-mono">
                    Resultado: <strong>0702</strong>
                  </div>
                </div>
              </div>
            </div>

            <!-- Características -->
            <div class="bg-gradient-to-r from-gray-800/80 to-gray-700/80 p-4 rounded-xl border border-cyan-400/30">
              <h4 class="font-semibold text-cyan-400 mb-2">🎯 Características:</h4>
              <ul class="text-sm text-gray-300 space-y-1">
                <li>• <strong class="text-green-400">Clave pseudoAleatoria</strong> con clave aleatoria única</li>
                <li>• <strong class="text-cyan-400">XOR bit a bit</strong> entre mensaje y clave</li>
                <li>• <strong class="text-yellow-400">Reversible:</strong> C ⊕ K = M original</li>
                <li>• <strong class="text-red-400">Clave única</strong> nunca se debe reutilizar</li>
              </ul>
            </div>

            <!-- Instrucciones -->
            <div class="bg-gradient-to-r from-gray-800/80 to-gray-700/80 p-4 rounded-xl border border-yellow-400/30">
              <h4 class="font-semibold text-yellow-400 mb-2">📋 Instrucciones:</h4>
              <div class="text-sm text-gray-300 space-y-2">
                <div><strong class="text-green-400">Para cifrar:</strong></div>
                <ol class="list-decimal ml-4 space-y-1">
                  <li>Escribe tu mensaje en texto plano</li>
                  <li>Genera o escribe una clave del mismo tamaño</li>
                  <li>Presiona "Cifrar"</li>
                  <li>Guarda el resultado hexadecimal y la clave</li>
                </ol>
                
                <div><strong class="text-cyan-400">Para descifrar:</strong></div>
                <ol class="list-decimal ml-4 space-y-1">
                  <li>Cambia a modo "Descifrar"</li>
                  <li>Pega el texto cifrado en hexadecimal</li>
                  <li>Ingresa la clave original</li>
                  <li>Presiona "Descifrar"</li>
                </ol>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</main>

<style>
  @keyframes fade-in {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  @keyframes slide-up {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  @keyframes shake {
    0%, 100% {
      transform: translateX(0);
    }
    25% {
      transform: translateX(-5px);
    }
    75% {
      transform: translateX(5px);
    }
  }
  
  .animate-fade-in {
    animation: fade-in 0.8s ease-out;
  }
  
  .animate-slide-up {
    animation: slide-up 0.5s ease-out;
  }
  
  .animate-shake {
    animation: shake 0.5s ease-in-out;
  }

  /* Glassmorphism */
  .backdrop-blur-sm {
    backdrop-filter: blur(8px);
  }

  /* Focus styles para tema hacker */
  button:focus {
    outline: 2px solid #22d3ee;
    outline-offset: 2px;
  }

  input:focus,
  textarea:focus {
    outline: none;
  }

  /* Loading animation */
  @keyframes spin {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
  
  .animate-spin {
    animation: spin 1s linear infinite;
  }

  /* Responsive */
  @media (max-width: 1024px) {
    .lg\:grid-cols-3 {
      grid-template-columns: 1fr;
    }
    .lg\:col-span-2 {
      grid-column: span 1;
    }
  }
</style>