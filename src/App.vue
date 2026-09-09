<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-avatar icon="build" color="white" text-color="primary" class="q-mr-sm" />
        <q-toolbar-title class="text-weight-bold">
          Servicio Técnico - Don Efraín
        </q-toolbar-title>
        <q-badge color="amber-8" text-color="black" label="Control de Taller" class="q-px-sm q-py-xs text-subtitle2" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md max-width-container">
        <div class="row q-col-gutter-md q-mb-lg items-center justify-between">
          <div class="col-12 col-sm-6 col-md-5">
            <q-input
              v-model="busqueda"
              dense
              outlined
              bg-color="white"
              placeholder="Buscar por cliente, marca, modelo o técnico..."
              clearable
            >
              <template v-slot:prepend>
                <q-icon name="search" />
              </template>
            </q-input>
          </div>
          <div class="col-12 col-sm-auto">
            <q-btn
              color="primary"
              icon="add_circle"
              label="Registrar Nuevo Servicio"
              size="md"
              class="full-width text-weight-bold"
              @click="abrirModalNuevo"
            />
          </div>
        </div>

        <div class="row q-gutter-sm q-mb-md">
          <q-chip icon="content_paste" color="grey-3" text-color="black">
            Total Registrados: <strong>{{ servicios.length }}</strong>
          </q-chip>
        </div>

        <div v-if="servicios.length === 0" class="text-center q-pa-xl bg-white rounded-borders shadow-1 q-my-md">
          <q-icon name="handyman" size="80px" color="grey-4" />
          <div class="text-h6 text-grey-7 q-mt-md">No hay registros de servicio técnico actualmente.</div>
          <div class="text-subtitle2 text-grey-5">Haz clic en "Registrar Nuevo Servicio" para crear uno.</div>
        </div>

        <div class="row q-col-gutter-md">
          <template v-for="(servicio, index) in servicios" :key="index">
            <div
              v-if="servicioCumpleFiltro(servicio)"
              class="col-12 col-sm-6 col-md-4"
            >
              <q-card
                flat
                bordered
                class="service-card transition-generic"
                :class="{
                  'card-pendiente': servicio.estadoPago === 'Pendiente',
                  'card-abono': servicio.estadoPago === 'Abono',
                  'card-pagado': servicio.estadoPago === 'Pagado',
                  'card-undelivered': servicio.estadoEquipo !== 'Entregado'
                }"
              >
                <q-card-section class="q-pb-xs">
                  <div class="row items-center no-wrap justify-between">
                    <div class="col ellipsis">
                      <div class="text-h6 text-weight-bold text-primary ellipsis">
                        {{ servicio.cliente }}
                      </div>
                      <div class="text-subtitle2 text-grey-9 row items-center q-gutter-xs">
                        <q-icon name="branding_watermark" color="primary" size="18px" />
                        <span><strong>Marca:</strong> {{ servicio.marca }}</span>
                      </div>
                      <div class="text-subtitle2 text-grey-9 row items-center q-gutter-xs">
                        <q-icon name="smartphone" color="primary" size="18px" />
                        <span><strong>Modelo:</strong> {{ servicio.modelo }}</span>
                      </div>
                    </div>
                  </div>
                </q-card-section>

                <q-separator />

                <q-card-section class="q-py-sm text-body2">
                  <div class="q-mb-xs">
                    <q-icon name="build" class="q-mr-xs" color="grey-7" />
                    <strong>Reparación:</strong> {{ servicio.tipoReparacion }}
                  </div>
                  <div class="q-mb-xs">
                    <q-icon name="person" class="q-mr-xs" color="grey-7" />
                    <strong>Técnico:</strong> {{ servicio.tecnico }}
                  </div>
                  <div class="q-mb-xs">
                    <q-icon name="event" class="q-mr-xs" color="grey-7" />
                    <strong>Fecha:</strong> {{ servicio.fechaHora }}
                  </div>

                  <div class="row items-center q-gutter-xs q-mt-sm">
                    <q-chip
                      dense
                      text-color="white"
                      :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)"
                    >
                      <q-avatar :icon="obtenerIconoEstadoEquipo(servicio.estadoEquipo)" />
                      {{ servicio.estadoEquipo }}
                    </q-chip>

                    <q-chip
                      dense
                      text-color="white"
                      :color="obtenerColorEstadoPago(servicio.estadoPago)"
                    >
                      <q-avatar :icon="obtenerIconoEstadoPago(servicio.estadoPago)" />
                      {{ servicio.estadoPago }}
                    </q-chip>
                  </div>

                  <div class="q-mt-sm bg-grey-2 q-pa-xs rounded-borders">
                    <div class="text-subtitle2 text-weight-bold">
                      Total: ${{ servicio.precio }} <span class="text-caption text-grey-7">({{ servicio.metodoPago }})</span>
                    </div>
                    <div class="text-caption text-positive text-weight-bold">
                      Abonado: ${{ servicio.abono || 0 }}
                    </div>
                    <div class="text-caption text-negative text-weight-bold">
                      Falta por pagar: ${{ calcularSaldoPendiente(servicio) }}
                    </div>
                  </div>

                  <div v-if="servicio.observaciones" class="q-mt-sm bg-grey-3 q-pa-xs rounded-borders text-caption text-grey-9">
                    <q-icon name="info" color="grey-7" class="q-mr-xs" />
                    <em>{{ servicio.observaciones }}</em>
                  </div>
                </q-card-section>

                <q-separator />

                <q-card-actions align="right" class="bg-grey-1">
                  <q-btn flat round dense color="primary" icon="edit" @click="abrirModalEditar(index)">
                    <q-tooltip>Editar Servicio</q-tooltip>
                  </q-btn>
                  <q-btn flat round dense color="negative" icon="delete" @click="confirmarEliminar(index)">
                    <q-tooltip>Eliminar Servicio</q-tooltip>
                  </q-btn>
                </q-card-actions>
              </q-card>
            </div>
          </template>
        </div>

        <q-dialog v-model="modalAbierto" persistent>
          <q-card style="min-width: 340px; max-width: 600px; width: 100%;">
            <q-card-section class="row items-center bg-primary text-white">
              <q-icon :name="modoEdicion ? 'edit' : 'add_circle'" size="28px" class="q-mr-sm" />
              <div class="text-h6 text-weight-bold">{{ modoEdicion ? 'Editar Servicio' : 'Nuevo Servicio Técnico' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-card-section class="q-pa-md">
              <q-form @submit="guardarServicio" class="q-gutter-sm">
                
                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del Cliente *"
                  outlined
                  dense
                  :rules="[val => (val && val.trim().length > 0) || 'El nombre del cliente es obligatorio']"
                />

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-6">
                    <q-input
                      v-model="formulario.marca"
                      label="Marca del Equipo *"
                      outlined
                      dense
                      placeholder="Ej: Apple, Samsung, Xiaomi"
                      :rules="[val => (val && val.trim().length > 0) || 'La marca es obligatoria']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-input
                      v-model="formulario.modelo"
                      label="Modelo del Equipo *"
                      outlined
                      dense
                      placeholder="Ej: iPhone 12, Galaxy A15, Redmi Note 10"
                      :rules="[val => (val && val.trim().length > 0) || 'El modelo es obligatorio']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.tipoReparacion"
                      :options="opcionesReparacion"
                      label="Tipo de Reparación *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione un tipo']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.tecnico"
                      :options="opcionesTecnicos"
                      label="Técnico *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione un técnico']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-4">
                    <q-input
                      v-model.number="formulario.precio"
                      type="number"
                      label="Precio Total ($) *"
                      outlined
                      dense
                      :rules="[val => (val !== null && val !== '' && val >= 0) || 'Ingrese un precio válido']"
                    />
                  </div>
                  <div class="col-12 col-sm-4">
                    <q-input
                      v-model.number="formulario.abono"
                      type="number"
                      label="Abono ($)"
                      outlined
                      dense
                      :rules="[val => (val === null || val === '' || val >= 0) || 'Abono inválido', val => val <= formulario.precio || 'El abono no puede superar el total']"
                    />
                  </div>
                  <div class="col-12 col-sm-4">
                    <q-input
                      :model-value="formulario.precio - (formulario.abono || 0)"
                      type="number"
                      label="Falta por Pagar ($)"
                      outlined
                      dense
                      readonly
                      bg-color="grey-2"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-4">
                    <q-select
                      v-model="formulario.metodoPago"
                      :options="opcionesMetodoPago"
                      label="Método de Pago *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione un método']"
                    />
                  </div>
                  <div class="col-12 col-sm-4">
                    <q-select
                      v-model="formulario.estadoPago"
                      :options="opcionesEstadoPago"
                      label="Estado del Pago *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione estado del pago']"
                    />
                  </div>
                  <div class="col-12 col-sm-4">
                    <q-select
                      v-model="formulario.estadoEquipo"
                      :options="opcionesEstadoEquipo"
                      label="Estado del Equipo *"
                      outlined
                      dense
                      :rules="[
                        val => !!val || 'Seleccione estado del equipo',
                        val => (val !== 'Entregado' || (formulario.precio - (formulario.abono || 0)) <= 0) || 'No se puede entregar si el equipo no está totalmente pagado'
                      ]"
                    />
                  </div>
                </div>

                <div v-if="(formulario.precio - (formulario.abono || 0)) > 0" class="text-caption text-negative text-weight-bold q-mb-xs">
                  * El equipo debe estar totalmente pagado para cambiar su estado a "Entregado".
                </div>

                <q-input
                  v-model="formulario.observaciones"
                  type="textarea"
                  label="Observaciones (Opcional)"
                  outlined
                  dense
                  rows="2"
                  placeholder="Ej: Pantalla partida en la esquina, no prende, respaldar fotos..."
                />

                <div class="row justify-end q-mt-md q-gutter-sm">
                  <q-btn label="Cancelar" color="grey-7" flat v-close-popup />
                  <q-btn type="submit" label="Guardar Servicio" color="primary" class="text-weight-bold" />
                </div>
              </q-form>
            </q-card-section>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalEliminarAbierto" persistent>
          <q-card style="min-width: 300px">
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="negative" text-color="white" class="q-mr-sm" />
              <span class="text-subtitle1">¿Está seguro de que desea eliminar este registro?</span>
            </q-card-section>
            <q-card-section class="text-caption text-grey-7 q-pt-none">
              Esta acción no se puede deshacer.
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="Cancelar" color="grey-7" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" class="text-weight-bold" @click="ejecutarEliminacion" v-close-popup />
            </q-card-actions>
          </q-card>
        </q-dialog>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, watch } from 'vue'

const CLAVE_STORAGE = 'taller_don_efrain_servicios_v3'

function cargarDeLocalStorage() {
  const datosGuardados = localStorage.getItem(CLAVE_STORAGE)
  if (datosGuardados) {
    try {
      return JSON.parse(datosGuardados)
    } catch (e) {
      return []
    }
  }
  return []
}

function guardarEnLocalStorage() {
  localStorage.setItem(CLAVE_STORAGE, JSON.stringify(servicios.value))
}

const servicios = ref(cargarDeLocalStorage())

const busqueda = ref('')
const modalAbierto = ref(false)
const modalEliminarAbierto = ref(false)
const modoEdicion = ref(false)
const indiceSeleccionado = ref(null)

const formulario = ref({
  cliente: '',
  marca: '',
  modelo: '',
  tipoReparacion: '',
  tecnico: '',
  precio: 0,
  abono: 0,
  metodoPago: '',
  estadoPago: '',
  estadoEquipo: '',
  observaciones: ''
})

const opcionesReparacion = [
  'Cambio de pantalla',
  'Cambio de batería',
  'Cambio de pin de carga',
  'Liberación',
  'Mantenimiento de software',
  'Cambio de flex',
  'Otros'
]

const opcionesTecnicos = ['Don Efraín', 'Técnico 1', 'Técnico 2']
const opcionesMetodoPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const opcionesEstadoPago = ['Pagado', 'Pendiente', 'Abono']
const opcionesEstadoEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

// Vigilante para revertir el estado del equipo si falta saldo
watch([() => formulario.value.precio, () => formulario.value.abono], ([nuevoPrecio, nuevoAbono]) => {
  const falta = (nuevoPrecio || 0) - (nuevoAbono || 0)
  if (falta > 0 && formulario.value.estadoEquipo === 'Entregado') {
    formulario.value.estadoEquipo = 'Listo para entregar'
  }
})

function obtenerFechaHoraActual() {
  const ahora = new Date()
  return ahora.toISOString().slice(0, 10) + ' ' + ahora.toTimeString().slice(0, 5)
}

function limpiarFormulario() {
  formulario.value = {
    cliente: '',
    marca: '',
    modelo: '',
    tipoReparacion: '',
    tecnico: '',
    precio: 0,
    abono: 0,
    metodoPago: 'Efectivo',
    estadoPago: 'Pendiente',
    estadoEquipo: 'Recibido',
    observaciones: ''
  }
}

function abrirModalNuevo() {
  modoEdicion.value = false
  limpiarFormulario()
  modalAbierto.value = true
}

function abrirModalEditar(index) {
  modoEdicion.value = true
  indiceSeleccionado.value = index
  const item = JSON.parse(JSON.stringify(servicios.value[index]))
  formulario.value = {
    ...item,
    abono: item.abono || 0
  }
  modalAbierto.value = true
}

function calcularSaldoPendiente(servicio) {
  const precio = servicio.precio || 0
  const abono = servicio.abono || 0
  const resto = precio - abono
  return resto > 0 ? resto : 0
}

function guardarServicio() {
  const falta = (formulario.value.precio || 0) - (formulario.value.abono || 0)
  
  // Bloqueo de seguridad preventivo
  if (falta > 0 && formulario.value.estadoEquipo === 'Entregado') {
    return
  }

  const datosLimpios = JSON.parse(JSON.stringify(formulario.value))
  datosLimpios.cliente = datosLimpios.cliente ? datosLimpios.cliente.trim() : ''
  datosLimpios.marca = datosLimpios.marca ? datosLimpios.marca.trim() : ''
  datosLimpios.modelo = datosLimpios.modelo ? datosLimpios.modelo.trim() : ''
  datosLimpios.abono = datosLimpios.abono || 0

  if (datosLimpios.abono >= datosLimpios.precio && datosLimpios.precio > 0) {
    datosLimpios.estadoPago = 'Pagado'
  } else if (datosLimpios.abono > 0) {
    datosLimpios.estadoPago = 'Abono'
  } else {
    datosLimpios.estadoPago = 'Pendiente'
  }

  if (modoEdicion.value) {
    servicios.value[indiceSeleccionado.value] = datosLimpios
  } else {
    datosLimpios.fechaHora = obtenerFechaHoraActual()
    servicios.value.push(datosLimpios)
  }
  guardarEnLocalStorage()
  modalAbierto.value = false
}

function confirmarEliminar(index) {
  indiceSeleccionado.value = index
  modalEliminarAbierto.value = true
}

function ejecutarEliminacion() {
  if (indiceSeleccionado.value !== null) {
    servicios.value.splice(indiceSeleccionado.value, 1)
    guardarEnLocalStorage()
    indiceSeleccionado.value = null
  }
}

function servicioCumpleFiltro(servicio) {
  if (!busqueda.value || !busqueda.value.trim()) return true
  const q = busqueda.value.trim().toLowerCase()
  return (
    (servicio.cliente && servicio.cliente.toLowerCase().includes(q)) ||
    (servicio.marca && servicio.marca.toLowerCase().includes(q)) ||
    (servicio.modelo && servicio.modelo.toLowerCase().includes(q)) ||
    (servicio.tecnico && servicio.tecnico.toLowerCase().includes(q))
  )
}

function obtenerColorEstadoEquipo(estado) {
  if (estado === 'Recibido') return 'blue-7'
  if (estado === 'En reparación') return 'deep-orange-7'
  if (estado === 'Listo para entregar') return 'green-7'
  if (estado === 'Entregado') return 'grey-7'
  return 'primary'
}

function obtenerIconoEstadoEquipo(estado) {
  if (estado === 'Recibido') return 'move_to_inbox'
  if (estado === 'En reparación') return 'build'
  if (estado === 'Listo para entregar') return 'task_alt'
  if (estado === 'Entregado') return 'verified_user'
  return 'help'
}

function obtenerColorEstadoPago(estado) {
  if (estado === 'Pagado') return 'positive'
  if (estado === 'Abono') return 'warning'
  if (estado === 'Pendiente') return 'negative'
  return 'grey'
}

function obtenerIconoEstadoPago(estado) {
  if (estado === 'Pagado') return 'check_circle'
  if (estado === 'Abono') return 'hourglass_bottom'
  if (estado === 'Pendiente') return 'error_outline'
  return 'help'
}
</script>

<style scoped>
.max-width-container {
  max-width: 1200px;
  margin: 0 auto;
}

.service-card {
  border-width: 2px;
  border-radius: 12px;
  overflow: hidden;
}

.card-pendiente {
  background-color: #fff5f5;
  border-color: #ef5350 !important;
}

.card-abono {
  background-color: #fffde7;
  border-color: #fbc02d !important;
}

.card-pagado {
  background-color: #ffffff;
  border-color: #66bb6a !important;
}

.card-undelivered {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}
</style>
