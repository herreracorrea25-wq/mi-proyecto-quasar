<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-tecnicos', [])
const mostrarFormulario = ref(false)
const mostrarEliminar = ref(false)
const mostrarFinalizar = ref(false)
const editar = ref(false)
const servicioSeleccionado = ref(null)
const busqueda = ref('')
const calificacion = ref(0)

const formulario = ref({
  id: null,
  cliente: '',
  equipo: '',
  reparacion: '',
  tecnico: '',
  fecha: '',
  precio: null,
  metodoPago: '',
  estadoPago: '',
  estadoEquipo: 'Recibido',
  calificacion: 0,
  observaciones: ''
})

const tecnicos = ['Don Efraín', 'Carlos', 'Miguel']
const reparaciones = ['Cambio de pantalla', 'Cambio de batería', 'Cambio de pin de carga', 'Liberación', 'Mantenimiento de software', 'Cambio de flex', 'Otros']
const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const estadosPago = ['Pagado', 'Pendiente', 'Abono']
const estadosEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

function limpiarFormulario() {
  formulario.value = { id: null, cliente: '', equipo: '', reparacion: '', tecnico: '', fecha: '', precio: null, metodoPago: '', estadoPago: '', estadoEquipo: 'Recibido', calificacion: 0, observaciones: '' }
}

function abrirFormulario() {
  limpiarFormulario()
  editar.value = false
  formulario.value.fecha = new Date().toISOString().slice(0, 16)
  mostrarFormulario.value = true
}

function cerrarFormulario() {
  mostrarFormulario.value = false
  limpiarFormulario()
}

function guardarServicio() {
  if (editar.value) {
    const index = servicios.value.findIndex(s => s.id === formulario.value.id)
    if (index !== -1) servicios.value[index] = { ...formulario.value }
  } else {
    servicios.value.push({ ...formulario.value, id: Date.now(), precio: Number(formulario.value.precio), calificacion: 0 })
  }
  cerrarFormulario()
}

function editarServicio(servicio) {
  formulario.value = { ...servicio }
  editar.value = true
  mostrarFormulario.value = true
}

function confirmarEliminar(servicio) {
  servicioSeleccionado.value = servicio
  mostrarEliminar.value = true
}

function eliminarServicio() {
  servicios.value = servicios.value.filter(s => s.id !== servicioSeleccionado.value.id)
  mostrarEliminar.value = false
  servicioSeleccionado.value = null
}

function cambiarEstado(servicio, estado) {
  servicio.estadoEquipo = estado
}

function abrirFinalizar(servicio) {
  servicioSeleccionado.value = servicio
  calificacion.value = 0
  mostrarFinalizar.value = true
}

function finalizarServicio() {
  if (!servicioSeleccionado.value || calificacion.value === 0) return
  servicioSeleccionado.value.estadoEquipo = 'Entregado'
  servicioSeleccionado.value.calificacion = calificacion.value
  mostrarFinalizar.value = false
  servicioSeleccionado.value = null
  calificacion.value = 0
}

function contarEstado(estado) {
  return servicios.value.filter(s => s.estadoEquipo === estado).length
}

function contarPago(estado) {
  return servicios.value.filter(s => s.estadoPago === estado).length
}

function promedio() {
  const calificados = servicios.value.filter(s => s.calificacion > 0)
  if (!calificados.length) return '0.0'
  const total = calificados.reduce((acc, s) => acc + s.calificacion, 0)
  return (total / calificados.length).toFixed(1)
}

function cantidadCalificaciones() {
  return servicios.value.filter(s => s.calificacion > 0).length
}

function buscar(servicio) {
  if (!busqueda.value) return true
  const texto = busqueda.value.toLowerCase()
  return servicio.cliente.toLowerCase().includes(texto) ||
    servicio.equipo.toLowerCase().includes(texto) ||
    servicio.reparacion.toLowerCase().includes(texto) ||
    servicio.tecnico.toLowerCase().includes(texto)
}

function precio(valor) {
  return new Intl.NumberFormat('es-CO', { style: 'currency', currency: 'COP', maximumFractionDigits: 0 }).format(valor || 0)
}

function fecha(valor) {
  return new Date(valor).toLocaleString('es-CO', { dateStyle: 'short', timeStyle: 'short' })
}

function colorPago(estado) {
  if (estado === 'Pagado') return 'positive'
  if (estado === 'Abono') return 'warning'
  return 'negative'
}

function iconoEstado(estado) {
  if (estado === 'Recibido') return 'inventory_2'
  if (estado === 'En reparación') return 'build'
  if (estado === 'Listo para entregar') return 'check_circle'
  return 'verified'
}

function colorEstado(estado) {
  if (estado === 'Recibido') return 'primary'
  if (estado === 'En reparación') return 'orange'
  if (estado === 'Listo para entregar') return 'positive'
  return 'grey'
}

function regla(valor) { return !!valor || 'Campo obligatorio' }
function reglaPrecio(valor) {
  if (valor === null || valor === '') return 'Obligatorio'
  if (Number(valor) < 0) return 'No negativo'
  return true
}
</script>

<template>
  <q-layout view="lHh Lpr lFf" class="bg-dark text-white">
    <q-header bordered class="bg-grey-10 text-white">
      <q-toolbar>
        <q-avatar color="primary" icon="build" text-color="white" size="32px" />
        <q-toolbar-title class="text-subtitle1 text-weight-bold">TecnoFix</q-toolbar-title>
        <q-btn dense color="primary" icon="add" label="Nuevo" unelevated @click="abrirFormulario" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        <div class="row q-col-gutter-sm q-mb-md">
          <div class="col-6 col-md-3"><q-card dark flat bordered class="q-pa-sm text-center"><div class="text-h6 text-bold">{{ servicios.length }}</div><div class="text-caption text-grey-5">Registrados</div></q-card></div>
          <div class="col-6 col-md-3"><q-card dark flat bordered class="q-pa-sm text-center"><div class="text-h6 text-bold text-orange">{{ contarEstado('En reparación') }}</div><div class="text-caption text-grey-5">En reparación</div></q-card></div>
          <div class="col-6 col-md-3"><q-card dark flat bordered class="q-pa-sm text-center"><div class="text-h6 text-bold text-positive">{{ contarEstado('Listo para entregar') }}</div><div class="text-caption text-grey-5">Listos</div></q-card></div>
          <div class="col-6 col-md-3"><q-card dark flat bordered class="q-pa-sm text-center"><div class="text-h6 text-bold text-amber">{{ promedio() }} ★</div><div class="text-caption text-grey-5">{{ cantidadCalificaciones() }} calif.</div></q-card></div>
        </div>

        <q-input dark dense v-model="busqueda" outlined clearable label="Buscar servicio" class="q-mb-md">
          <template v-slot:prepend><q-icon name="search" /></template>
        </q-input>

        <div v-if="servicios.length" class="row q-col-gutter-sm">
          <div v-for="servicio in servicios" :key="servicio.id" class="col-12 col-md-4">
            <q-card dark flat bordered v-if="buscar(servicio)" class="column justify-between q-pa-sm">
              <div>
                <div class="row justify-between items-center q-mb-xs">
                  <span class="text-caption text-grey-5">#{{ servicio.id }}</span>
                  <q-badge dense :color="colorPago(servicio.estadoPago)">{{ servicio.estadoPago }}</q-badge>
                </div>
                <div class="text-subtitle1 text-bold">{{ servicio.cliente }}</div>
                <div class="text-caption text-primary">{{ servicio.equipo }} - {{ servicio.reparacion }}</div>
                <q-separator dark class="q-my-xs" />
                <div class="text-caption text-grey-4"><q-icon :name="iconoEstado(servicio.estadoEquipo)" :color="colorEstado(servicio.estadoEquipo)" /> {{ servicio.estadoEquipo }}</div>
                <div class="text-caption text-grey-4"><q-icon name="person" /> {{ servicio.tecnico }} | <q-icon name="payments" /> {{ precio(servicio.precio) }}</div>
                <div v-if="servicio.observaciones" class="text-caption text-grey-5 q-mt-xs">Obs: {{ servicio.observaciones }}</div>
                <div v-if="servicio.estadoEquipo === 'Entregado'" class="q-mt-xs">
                  <q-rating :model-value="servicio.calificacion" max="5" color="amber" readonly size="1em" />
                </div>
              </div>
              <q-card-actions align="right" class="q-pa-none q-mt-sm">
                <q-btn v-if="servicio.estadoEquipo === 'Recibido'" dense size="sm" color="orange" icon="build" label="Iniciar" @click="cambiarEstado(servicio, 'En reparación')" />
                <q-btn v-else-if="servicio.estadoEquipo === 'En reparación'" dense size="sm" color="positive" icon="check" label="Listo" @click="cambiarEstado(servicio, 'Listo para entregar')" />
                <q-btn v-else-if="servicio.estadoEquipo === 'Listo para entregar'" dense size="sm" color="positive" icon="done_all" label="Entregar" @click="abrirFinalizar(servicio)" />
                <q-btn dense flat size="sm" color="primary" icon="edit" @click="editarServicio(servicio)" />
                <q-btn dense flat size="sm" color="negative" icon="delete" @click="confirmarEliminar(servicio)" />
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <div v-else class="text-center q-pa-xl text-grey-5">
          <q-icon name="phone_android" size="48px" />
          <div class="text-subtitle1">No hay servicios registrados</div>
        </div>
      </q-page>
    </q-page-container>

    <q-dialog v-model="mostrarFormulario" persistent>
      <q-card dark class="q-pa-md" style="width: 400px; max-width: 90vw;">
        <div class="text-subtitle1 text-bold q-mb-sm">{{ editar ? 'Editar' : 'Nuevo' }} servicio</div>
        <q-form @submit.prevent="guardarServicio" class="q-gutter-sm">
          <q-input dark dense v-model="formulario.cliente" label="Cliente *" outlined :rules="[regla]" />
          <q-input dark dense v-model="formulario.equipo" label="Equipo *" outlined :rules="[regla]" />
          <q-select dark dense v-model="formulario.reparacion" :options="reparaciones" label="Reparación *" outlined :rules="[regla]" />
          <q-select dark dense v-model="formulario.tecnico" :options="tecnicos" label="Técnico *" outlined :rules="[regla]" />
          <q-input dark dense v-model="formulario.fecha" type="datetime-local" label="Fecha *" outlined :rules="[regla]" />
          <q-input dark dense v-model.number="formulario.precio" type="number" label="Precio *" prefix="$" outlined :rules="[reglaPrecio]" />
          <q-select dark dense v-model="formulario.metodoPago" :options="metodosPago" label="Método pago *" outlined :rules="[regla]" />
          <q-select dark dense v-model="formulario.estadoPago" :options="estadosPago" label="Estado pago *" outlined :rules="[regla]" />
          <q-select dark dense v-model="formulario.estadoEquipo" :options="estadosEquipo" label="Estado equipo *" outlined :rules="[regla]" />
          <q-input dark dense v-model="formulario.observaciones" type="textarea" label="Observaciones" outlined autogrow />
          <div class="row justify-end q-mt-md q-gutter-sm">
            <q-btn dense flat label="Cancelar" @click="cerrarFormulario" />
            <q-btn dense color="primary" label="Guardar" type="submit" unelevated />
          </div>
        </q-form>
      </q-card>
    </q-dialog>

    <q-dialog v-model="mostrarFinalizar" persistent>
      <q-card dark class="q-pa-md text-center" style="width: 300px;">
        <div class="text-subtitle1 text-bold q-mb-sm">Calificar servicio</div>
        <q-rating v-model="calificacion" max="5" color="amber" size="2em" />
        <div class="row justify-end q-mt-md q-gutter-sm">
          <q-btn dense flat label="Cancelar" @click="mostrarFinalizar = false" />
          <q-btn dense color="positive" label="Confirmar" :disable="!calificacion" @click="finalizarServicio" />
        </div>
      </q-card>
    </q-dialog>

    <q-dialog v-model="mostrarEliminar">
      <q-card dark class="q-pa-md text-center" style="width: 300px;">
        <div class="text-subtitle1 text-bold q-mb-sm">¿Eliminar servicio?</div>
        <div class="row justify-end q-mt-md q-gutter-sm">
          <q-btn dense flat label="Cancelar" @click="mostrarEliminar = false" />
          <q-btn dense color="negative" label="Eliminar" @click="eliminarServicio" />
        </div>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<style>
body { background: #121212; color: #e0e0e0; }
</style>