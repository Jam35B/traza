<template>
  <!-- Se define la estructura de la página, usando Quasar's q-page para el diseño. -->
  <q-page>
    <!-- Título centrado de la aplicación. -->
    <p class="text-h6 text-center">Traza</p>
    <!-- Botón centrado que inicia la lectura de códigos de barras al hacer clic. -->
    <div class="text-center">
      <q-btn
        label="Iniciar Lectura"
        color="primary"
        icon="mdi-barcode-scan"
        size="lg"
        @click="initReader" 
      /> 
    </div> <!-- Evento que llama a la función initReader al hacer clic. -->

    <!-- Muestra el código de barras leído si la variable 'code' tiene un valor. -->
    <p
      v-if="code"
      class="text-h6 text-center q-mt-lg"> <!-- Condición que muestra el código solo si ya fue escaneado. -->
      Código Leido: {{ code }} <!-- Muestra el código leído. -->

      <!-- Botón que copia el código de barras leído al portapapeles. -->
      <q-btn
        icon="mdi-content-copy"
        @click="copyCode" 
        size="md"
        dense
        color="blue-9"
      /> <!-- Llama a la función copyCode al hacer clic. -->
    </p>

    <!-- Div que muestra el lector de la cámara cuando 'cameraStatus' es verdadero. -->
    <div
      class="text-center"
      v-show="cameraStatus" 
      id="reader"></div> <!-- Controla la visibilidad según el estado de la cámara y define el area donde se visualiza el flujo de la camara -->
  </q-page>
</template>

<script>
import { defineComponent, ref } from 'vue' // Importa Vue para definir componentes y crear referencias reactivas.
import Quagga from 'quagga' // Importa la librería Quagga para escanear códigos de barras.
import { copyToClipboard, useQuasar } from 'quasar' // Importa utilidades de Quasar para copiar al portapapeles y notificaciones.

export default defineComponent({
  name: 'IndexPage', // Nombre del componente.
  setup () {
    // Definición de variables reactivas.
    const code = ref('') // Almacena el código escaneado.
    const cameraStatus = ref(false) // Controla el estado de la cámara (activo o inactivo).
    const q$ = useQuasar() // Acceso a las funcionalidades de Quasar, como notificaciones.

    // Función que inicializa el lector de códigos de barras usando Quagga.
    const initReader = () => {
      cameraStatus.value = true // Activa la cámara.
      Quagga.init({
        inputStream: {
          name: 'Live', // Fuente de entrada en vivo (cámara).
          type: 'LiveStream',
          target: document.querySelector('#reader') // El elemento donde se mostrará la transmisión de la cámara.
        },
        decoder: {
          readers: ['ean_reader'] // Especifica el tipo de códigos de barras a leer (EAN).
        }
      }, function (err) {
        if (err) {
          console.log(err) // Muestra errores en la consola.
          return
        }
        console.log('Initialization finished. Ready to start')
        Quagga.start() // Inicia el escaneo.
        Quagga.onDetected((data) => {
          onDetected(data) // Llama a la función onDetected cuando se detecta un código.
        })
      })
    }

    // Función que detiene el lector de códigos de barras.
    const stopReader = () => {
      cameraStatus.value = false // Desactiva la cámara.
      Quagga.stop() // Detiene el lector.
    }

    // Función que se ejecuta cuando se detecta un código de barras.
    const onDetected = (data) => {
      code.value = data.codeResult.code // Asigna el código escaneado a la variable 'code'.
      stopReader() // Detiene el lector tras detectar un código.
      console.log(data) // Muestra los datos detectados en la consola.
    }

    // Función para copiar el código escaneado al portapapeles.
    const copyCode = () => {
      copyToClipboard(code.value) // Copia el valor de 'code' al portapapeles.
        .then(() => {
          q$.notify({
            message: 'Código de barras copiado con éxito!', // Notificación de éxito.
            type: 'positivo'
          })
        })
        .catch(() => {
          q$.notify({
            message: 'error al copiar el codigo de barras!', // Notificación de error.
            type: 'negative'
          })
        })
    }

    // Retorno de las funciones y variables para que estén disponibles en el template.
    return {
      initReader,
      stopReader,
      cameraStatus,
      code,
      copyCode
    }
  }
})
</script>
