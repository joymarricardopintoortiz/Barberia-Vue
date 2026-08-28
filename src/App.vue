<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('barberia-servicios', [])

const mostrarModal = ref(false)
const modoEdicion = ref(false)
const idEditando = ref(null)
const mensajeError = ref('')

const servicioActual = ref({
  nombre: '',
  servicio: '',
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  metodoPago: '',
  estadoPago: 'pagado',
  calificacion: 5,
  observaciones: ''
})

function limpiarFormulario() {
  servicioActual.value = {
    nombre: '',
    servicio: '',
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodoPago: '',
    estadoPago: 'pagado',
    calificacion: 5,
    observaciones: ''
  }

  mensajeError.value = ''
}

function abrirModalNuevo() {
  limpiarFormulario()
  modoEdicion.value = false
  idEditando.value = null
  mostrarModal.value = true
}

function abrirModalEditar(servicio) {
  servicioActual.value = {
    nombre: servicio.nombre,
    servicio: servicio.servicio,
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    metodoPago: servicio.metodoPago,
    estadoPago: servicio.estadoPago,
    calificacion: servicio.calificacion,
    observaciones: servicio.observaciones
  }

  idEditando.value = servicio.id
  modoEdicion.value = true
  mensajeError.value = ''
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
  mensajeError.value = ''
}

function validarFormulario() {
  if (!servicioActual.value.nombre.trim()) {
    mensajeError.value = 'Ingrese el nombre del cliente.'
    return false
  }

  if (!servicioActual.value.servicio) {
    mensajeError.value = 'Seleccione el tipo de servicio.'
    return false
  }

  if (!servicioActual.value.barbero) {
    mensajeError.value = 'Seleccione el barbero.'
    return false
  }

  if (!servicioActual.value.fecha) {
    mensajeError.value = 'Seleccione la fecha.'
    return false
  }

  if (!servicioActual.value.hora) {
    mensajeError.value = 'Seleccione la hora.'
    return false
  }

  if (!servicioActual.value.precio) {
    mensajeError.value = 'Ingrese el precio.'
    return false
  }

  if (Number(servicioActual.value.precio) <= 0) {
    mensajeError.value = 'El precio debe ser mayor que 0.'
    return false
  }

  if (!servicioActual.value.metodoPago) {
    mensajeError.value = 'Seleccione el método de pago.'
    return false
  }

  if (!servicioActual.value.estadoPago) {
    mensajeError.value = 'Seleccione el estado del pago.'
    return false
  }

  if (
    Number(servicioActual.value.calificacion) < 1 ||
    Number(servicioActual.value.calificacion) > 5
  ) {
    mensajeError.value = 'La calificación debe estar entre 1 y 5.'
    return false
  }

  return true
}

function guardarServicio() {
  mensajeError.value = ''

  if (!validarFormulario()) {
    return
  }

  if (modoEdicion.value) {
    const posicion = servicios.value.findIndex(
      servicio => servicio.id === idEditando.value
    )

    if (posicion !== -1) {
      servicios.value[posicion] = {
        id: idEditando.value,
        nombre: servicioActual.value.nombre.trim(),
        servicio: servicioActual.value.servicio,
        barbero: servicioActual.value.barbero,
        fecha: servicioActual.value.fecha,
        hora: servicioActual.value.hora,
        precio: Number(servicioActual.value.precio),
        metodoPago: servicioActual.value.metodoPago,
        estadoPago: servicioActual.value.estadoPago,
        calificacion: Number(servicioActual.value.calificacion),
        observaciones: servicioActual.value.observaciones.trim()
      }
    }
  } else {
    const nuevoServicio = {
      id: Date.now(),
      nombre: servicioActual.value.nombre.trim(),
      servicio: servicioActual.value.servicio,
      barbero: servicioActual.value.barbero,
      fecha: servicioActual.value.fecha,
      hora: servicioActual.value.hora,
      precio: Number(servicioActual.value.precio),
      metodoPago: servicioActual.value.metodoPago,
      estadoPago: servicioActual.value.estadoPago,
      calificacion: Number(servicioActual.value.calificacion),
      observaciones: servicioActual.value.observaciones.trim()
    }

    servicios.value.unshift(nuevoServicio)
  }

  cerrarModal()
}

function eliminarServicio(id) {
  const confirmar = confirm(
    '¿Está seguro de que desea eliminar este servicio?'
  )

  if (!confirmar) {
    return
  }

  const posicion = servicios.value.findIndex(
    servicio => servicio.id === id
  )

  if (posicion !== -1) {
    servicios.value.splice(posicion, 1)
  }
}

function obtenerIconoPago(metodo) {
  if (metodo === 'efectivo') {
    return '💵'
  }

  if (metodo === 'transferencia') {
    return '📱'
  }

  if (metodo === 'tarjeta') {
    return '💳'
  }

  return '💰'
}

function obtenerClaseEstado(estado) {
  if (estado === 'pagado') {
    return 'estado-pagado'
  }

  if (estado === 'pendiente') {
    return 'estado-pendiente'
  }

  if (estado === 'fiado') {
    return 'estado-fiado'
  }

  return ''
}

function obtenerClaseCalificacion(calificacion) {
  if (calificacion <= 2) {
    return 'calificacion-baja'
  }

  if (calificacion === 3) {
    return 'calificacion-media'
  }

  return 'calificacion-alta'
}

function obtenerEstrellas(calificacion) {
  let estrellas = ''

  for (let i = 1; i <= 5; i++) {
    if (i <= calificacion) {
      estrellas += '★'
    } else {
      estrellas += '☆'
    }
  }

  return estrellas
}

function formatearPrecio(precio) {
  return Number(precio).toLocaleString('es-CO')
}

function calcularVentas() {
  let total = 0

  for (let servicio of servicios.value) {
    total += Number(servicio.precio)
  }

  return total.toLocaleString('es-CO')
}

function calcularPorCobrar() {
  let total = 0

  for (let servicio of servicios.value) {
    if (servicio.estadoPago !== 'pagado') {
      total += Number(servicio.precio)
    }
  }

  return total.toLocaleString('es-CO')
}
</script>

<template>
  <div class="app">

    <header class="header">
      <div>
        <p class="etiqueta">BARBERÍA</p>
        <h1>Barbería Don Ramiro</h1>
        <p class="subtitulo">
          Registro de servicios y clientes
        </p>
      </div>

      <button class="btn-principal" @click="abrirModalNuevo">
        + Nuevo servicio
      </button>
    </header>

    <section class="resumen">

      <div class="resumen-card">
        <span class="resumen-icono">📋</span>
        <div>
          <span>Total servicios</span>
          <strong>{{ servicios.length }}</strong>
        </div>
      </div>

      <div class="resumen-card">
        <span class="resumen-icono">💰</span>
        <div>
          <span>Ventas registradas</span>
          <strong>
            ${{ servicios.reduce((total, item) => total + Number(item.precio), 0).toLocaleString('es-CO') }}
          </strong>
        </div>
      </div>

      <div class="resumen-card">
        <span class="resumen-icono">⏳</span>
        <div>
          <span>Por cobrar</span>
          <strong>
            ${{ servicios
              .filter(item => item.estadoPago !== 'pagado')
              .reduce((total, item) => total + Number(item.precio), 0)
              .toLocaleString('es-CO') }}
          </strong>
        </div>
      </div>

    </section>

    <main class="contenido">

      <div class="titulo-lista">
        <div>
          <h2>Servicios registrados</h2>
          <p>
            Aquí aparecen todos los servicios realizados.
          </p>
        </div>
      </div>

      <div v-if="servicios.length === 0" class="vacio">
        <div class="vacio-icono">✂️</div>
        <h3>No hay servicios registrados</h3>
        <p>
          Registra el primer servicio de la barbería.
        </p>

        <button class="btn-principal" @click="abrirModalNuevo">
          Registrar servicio
        </button>
      </div>

      <div v-else class="lista">

        <article
          v-for="servicio in servicios"
          :key="servicio.id"
          class="servicio-card"
          :class="obtenerClaseEstado(servicio.estadoPago)"
        >

          <div class="card-header">

            <div class="cliente">
              <div class="avatar">
                {{ servicio.nombre.charAt(0).toUpperCase() }}
              </div>

              <div>
                <h3>{{ servicio.nombre }}</h3>

                <p>
                  {{ servicio.fecha }}
                  ·
                  {{ servicio.hora }}
                </p>
              </div>
            </div>

            <span
              class="estado"
              :class="obtenerClaseEstado(servicio.estadoPago)"
            >
              <span v-if="servicio.estadoPago === 'pagado'">Pagado</span>
              <span v-else-if="servicio.estadoPago === 'pendiente'">Pendiente</span>
              <span v-else>Fiado</span>
            </span>

          </div>

          <div class="card-body">

            <div class="dato">
              <span class="dato-label">Servicio</span>
              <strong>{{ servicio.servicio }}</strong>
            </div>

            <div class="dato">
              <span class="dato-label">Barbero</span>
              <strong>{{ servicio.barbero }}</strong>
            </div>

            <div class="dato">
              <span class="dato-label">Precio</span>
              <strong class="precio">
                ${{ formatearPrecio(servicio.precio) }}
              </strong>
            </div>

            <div class="dato">
              <span class="dato-label">Pago</span>
              <strong>
                {{ obtenerIconoPago(servicio.metodoPago) }}
                {{ servicio.metodoPago }}
              </strong>
            </div>

            <div class="dato">
              <span class="dato-label">Calificación</span>

              <strong
                class="estrellas"
                :class="obtenerClaseCalificacion(servicio.calificacion)"
              >
                {{ obtenerEstrellas(servicio.calificacion) }}
              </strong>
            </div>

          </div>

          <div
            v-if="servicio.calificacion <= 2"
            class="alerta-calificacion"
          >
            Calificación baja. Conviene revisar la experiencia del cliente.
          </div>

          <div
            v-if="servicio.estadoPago === 'fiado'"
            class="alerta-fiado"
          >
            Este servicio quedó fiado. Hay dinero pendiente por cobrar.
          </div>

          <div
            v-if="servicio.observaciones"
            class="observaciones"
          >
            <strong>Observaciones:</strong>
            {{ servicio.observaciones }}
          </div>

          <div class="card-footer">

            <span class="metodo">
              {{ obtenerIconoPago(servicio.metodoPago) }}
              {{ servicio.metodoPago }}
            </span>

            <div class="acciones">
              <button
                class="btn-editar"
                @click="abrirModalEditar(servicio)"
              >
                Editar
              </button>

              <button
                class="btn-eliminar"
                @click="eliminarServicio(servicio.id)"
              >
                Eliminar
              </button>
            </div>

          </div>

        </article>

      </div>
    </main>

    <div
      v-show="mostrarModal"
      class="modal-fondo"
      @click.self="cerrarModal"
    >

      <div class="modal">

        <div class="modal-header">
          <div>
            <p class="etiqueta">REGISTRO</p>

            <h2>
              {{ modoEdicion ? 'Editar servicio' : 'Nuevo servicio' }}
            </h2>
          </div>

          <button
            class="btn-cerrar"
            @click="cerrarModal"
          >
            ×
          </button>
        </div>

        <div
          v-if="mensajeError"
          class="error"
        >
          {{ mensajeError }}
        </div>

        <form @submit.prevent="guardarServicio">

          <div class="form-grid">

            <div class="campo campo-completo">
              <label>Nombre del cliente *</label>

              <input
                v-model="servicioActual.nombre"
                type="text"
                placeholder="Ej: Carlos Pérez"
              >
            </div>

            <div class="campo">
              <label>Tipo de servicio *</label>

              <select v-model="servicioActual.servicio">
                <option value="">
                  Seleccione...
                </option>

                <option value="Corte clásico">
                  Corte clásico
                </option>

                <option value="Corte moderno">
                  Corte moderno
                </option>

                <option value="Barba">
                  Barba
                </option>

                <option value="Corte + barba">
                  Corte + barba
                </option>

                <option value="Cejas">
                  Cejas
                </option>

                <option value="Tinte">
                  Tinte
                </option>

                <option value="Otro">
                  Otro
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Barbero *</label>

              <select v-model="servicioActual.barbero">
                <option value="">
                  Seleccione...
                </option>

                <option value="Don Ramiro">
                  Don Ramiro
                </option>

                <option value="Carlos">
                  Carlos
                </option>

                <option value="Andrés">
                  Andrés
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Fecha *</label>

              <input
                v-model="servicioActual.fecha"
                type="date"
              >
            </div>

            <div class="campo">
              <label>Hora *</label>

              <input
                v-model="servicioActual.hora"
                type="time"
              >
            </div>

            <div class="campo">
              <label>Precio cobrado *</label>

              <input
                v-model="servicioActual.precio"
                type="number"
                min="1"
                placeholder="25000"
              >
            </div>

            <div class="campo">
              <label>Método de pago *</label>

              <select v-model="servicioActual.metodoPago">
                <option value="">
                  Seleccione...
                </option>

                <option value="efectivo">
                  Efectivo
                </option>

                <option value="transferencia">
                  Transferencia
                </option>

                <option value="tarjeta">
                  Tarjeta
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Estado del pago *</label>

              <select v-model="servicioActual.estadoPago">
                <option value="pagado">
                  Pagado
                </option>

                <option value="pendiente">
                  Pendiente
                </option>

                <option value="fiado">
                  Fiado
                </option>
              </select>
            </div>

            <div class="campo campo-completo">
              <label>Calificación del cliente *</label>

              <div class="calificacion-form">

                <button
                  v-for="numero in 5"
                  :key="numero"
                  type="button"
                  class="estrella-btn"
                  :class="{ seleccionada: numero <= servicioActual.calificacion }"
                  @click="servicioActual.calificacion = numero"
                >
                  ★
                </button>

                <span>
                  {{ servicioActual.calificacion }}/5
                </span>

              </div>
            </div>

            <div class="campo campo-completo">
              <label>Observaciones</label>

              <textarea
                v-model="servicioActual.observaciones"
                rows="3"
                placeholder="Ej: Cliente pidió degradado bajo..."
              ></textarea>
            </div>

          </div>

          <div class="modal-footer">

            <button
              type="button"
              class="btn-secundario"
              @click="cerrarModal"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="btn-principal"
            >
              {{ modoEdicion ? 'Guardar cambios' : 'Registrar servicio' }}
            </button>

          </div>

        </form>

      </div>
    </div>

  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f4f1eb;
  color: #242424;
}

button, input, select, textarea {
  font: inherit;
}

button {
  cursor: pointer;
}

.app {
  min-height: 100vh;
}

.header {
  background: #171717;
  color: white;
  padding: 28px 5%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
}

.header h1 {
  margin: 4px 0;
  font-size: 28px;
}

.subtitulo {
  margin: 0;
  color: #bcbcbc;
  font-size: 15px;
}

.etiqueta {
  margin: 0;
  color: #c89b5c;
  font-size: 12px;
  font-weight: bold;
  letter-spacing: 2px;
}

.btn-principal {
  border: 0;
  background: #c89b5c;
  color: #171717;
  padding: 12px 18px;
  border-radius: 10px;
  font-weight: bold;
}

.btn-secundario {
  border: 1px solid #ddd;
  background: white;
  color: #333;
  padding: 11px 18px;
  border-radius: 10px;
  font-weight: bold;
}

.btn-editar, .btn-eliminar {
  border: 0;
  padding: 8px 12px;
  border-radius: 8px;
  font-weight: bold;
}

.btn-editar {
  background: #eee9df;
  color: #57452f;
}

.btn-eliminar {
  background: #fbe4e2;
  color: #a63c32;
}

.btn-cerrar {
  border: 0;
  background: transparent;
  font-size: 30px;
  line-height: 1;
  color: #777;
  padding: 0;
}

.resumen {
  max-width: 1200px;
  margin: 25px auto;
  padding: 0 20px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px
}

.resumen-card {
  background: white;
  border-radius: 14px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  box-shadow: 0 3px 15px rgba(0,0,0,0.05);
  border: 1px solid #eee;
}

.resumen-card > span:not(.resumen-icono) {
  color: #777;
}

.resumen-card strong {
  display: block;
  margin-top: 5px;
  font-size: 22px;
  color: #222;
}

.resumen-icono {
  font-size: 28px;
  min-width: 35px;
  text-align: center;
}

.contenido {
  max-width: 1200px;
  margin: auto;
  padding: 0 20px 50px;
}

.titulo-lista {
  margin-bottom: 20px;
}

.titulo-lista h2 {
  margin: 0 0 5px;
  font-size: 25px;
}

.titulo-lista p {
  margin: 0;
  color: #777;
}

.lista {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 18px;
}

.servicio-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 18px rgba(0,0,0,0.06);
  border-left: 5px solid #c89b5c;
}

.servicio-card .estado-pendiente {
  border-left-color: #e0a72f;
}

.servicio-card .estado-fiado {
  border-left-color: #c7483d;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 10px;
}

.cliente {
  display: flex;
  gap: 12px;
  align-items: center;
  min-width: 0;
}

.avatar {
  width: 44px;
  height: 44px;
  flex-shrink: 0;
  background: #171717;
  color: #c89b5c;
  border-radius: 50%;
  display: grid;
  place-items: center;
  font-weight: bold;
  font-size: 18px;
}

.cliente h3 {
  margin: 0;
  font-size: 17px;
  word-break: break-word;
}

.cliente p {
  margin: 4px 0 0;
  color: #888;
  font-size: 13px;
}

.estado {
  padding: 6px 9px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
  white-space: nowrap;
}

.estado-pagado {
  background: #e6f5e8;
  color: #287a39;
}

.estado-pendiente {
  background: #fff2cf;
  color: #966b00;
}

.estado-fiado {
  background: #fff2cf;
  color: #a33a31;
}

.card-body {
  margin-top: 20px;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.dato {
  min-width: 0;
}

.dato-label {
  display: block;
  color: #888;
  font-size: 12px;
  margin-bottom: 4px;
}

.dato strong {
  display: block;
  word-break: break-word;
}

.precio {
  color: #9a713b;
  font-size: 18px;
}

.estrellas {
  letter-spacing: 2px;
  font-size: 17px;
  white-space: nowrap;
}

.calificacion-baja {
  color: #c0392b;
}

.calificacion-media {
  color: #d69a20;
}

.calificacion-alta {
  color: #c89b5c;
}

.alerta-calificacion, .alerta-fiado {
  margin-top: 15px;
  padding: 10px;
  border-radius: 8px;
  font-size: 13px;
  line-height: 1.4;
}

.alerta-calificacion {
  background: #fff6dc;
  color: #806000;
}

.observaciones {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #eee;
  font-size: 13px;
  color: #666;
  line-height: 1.5;
  word-break: break-word;
}

.card-footer {
  margin-top: 18px;
  padding-top: 15px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
}

.metodo {
  font-size: 13px;
  color: #666;
}

.acciones {
  display: flex;
  gap: 8px;
}

.vacio {
  background: white;
  padding: 60px 20px;
  text-align: center;
  border-radius: 16px;
  box-shadow: 0 4px 18px rgba(0,0,0,0.05);
}

.vacio-icono {
  font-size: 45px;
  margin-bottom: 10px;
}

.vacio h3 {
  margin: 0 0 5px;
}

.vacio p {
  color: #777;
  margin-bottom: 20px;
}


</style>