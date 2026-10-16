<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import './style.css'

const dolares = ref([])
const tipoDolar = ref('blue')
const operacion = ref('comprar')
const monto = ref(10000)
const resultado = ref(null)
const error = ref('')
const cargando = ref(false)

// Obtener los valores del dolar desde la API
async function mostrarDolares() {
  cargando.value = true
  error.value = ''

  try {
    const respuesta = await axios.get(
      'https://dolarapi.com/v1/dolares'
    )

    dolares.value = respuesta.data
  } catch (e) {
    error.value = 'No se pudieron cargar los dolares.'
  } finally {
    cargando.value = false
  }
}

// Calcular la compra o venta
function calcular() {
  const dolar = dolares.value.find(
    d => d.casa === tipoDolar.value
  )

  if (
    !dolar ||
    monto.value === '' ||
    monto.value === null ||
    !Number.isFinite(Number(monto.value)) ||
    Number(monto.value) <= 0
  ) {
    resultado.value = null
    return
  }

  if (operacion.value === 'comprar') {
    // Pesos a dolares: usamos el precio de venta
    resultado.value =
      Number(monto.value) / Number(dolar.venta)
  } else {
    // Dolares a pesos: usamos el precio de compra
    resultado.value =
      Number(monto.value) * Number(dolar.compra)
  }
}

// Ejecutar cuando se carga la pagina
onMounted(mostrarDolares)
</script>

<template>
  <div class="contenedor">
    <header>
      <h1>Compra y venta de dolares</h1>
      <p class="subtitulo">
        Cotizaciones del dolar en Argentina
      </p>
    </header>

    <section class="panel">
      <h2>Cotizaciones actuales</h2>

      <button
        @click="mostrarDolares"
        :disabled="cargando"
      >
        {{ cargando ? 'Cargando...' : 'Actualizar cotizaciones' }}
      </button>

      <p v-if="error" class="error">{{ error }}</p>

      <p v-if="cargando && dolares.length === 0">
        Cargando cotizaciones...
      </p>

      <table v-if="dolares.length > 0">
        <thead>
          <tr>
            <th>Tipo de dolar</th>
            <th>Compra</th>
            <th>Venta</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="dolar in dolares"
            :key="dolar.casa"
          >
            <td>{{ dolar.nombre }}</td>
            <td>
              {{ dolar.compra != null
                ? '$ ' + Number(dolar.compra).toLocaleString('es-AR')
                : 'Sin dato' }}
            </td>
            <td>
              {{ dolar.venta != null
                ? '$ ' + Number(dolar.venta).toLocaleString('es-AR')
                : 'Sin dato' }}
            </td>
          </tr>
        </tbody>
      </table>
    </section>

    <section class="panel">
      <h2>Convertidor de monedas</h2>

      <label>Operacion:</label>

      <select v-model="operacion" @change="calcular">
        <option value="comprar">
          Comprar dolares con pesos
        </option>
        <option value="vender">
          Vender dolares por pesos
        </option>
      </select>

      <label>Tipo de dolar:</label>

      <select v-model="tipoDolar" @change="calcular">
        <option
          v-for="dolar in dolares"
          :key="dolar.casa"
          :value="dolar.casa"
        >
          {{ dolar.nombre }}
        </option>
      </select>

      <label>
        {{ operacion === 'comprar'
          ? 'Cantidad de pesos:'
          : 'Cantidad de dolares:' }}
      </label>

      <input
        type="number"
        v-model.number="monto"
        min="0"
        placeholder="Ingrese un monto"
        @input="resultado = null"
      />

      <button @click="calcular">
        Calcular
      </button>

      <div v-if="resultado !== null" class="resultado">
        <p>Recibiras aproximadamente:</p>

        <h2>
          {{ operacion === 'comprar' ? 'US$ ' : '$ ' }}
          {{ resultado.toLocaleString('es-AR', {
            minimumFractionDigits: 2,
            maximumFractionDigits: 2
          }) }}
        </h2>

        <p>
          Se utiliza la cotizacion de
          {{ operacion === 'comprar' ? 'venta' : 'compra' }}.
        </p>

        <small>
          El calculo es orientativo y no incluye comisiones.
        </small>
      </div>
    </section>
  </div>
</template>
