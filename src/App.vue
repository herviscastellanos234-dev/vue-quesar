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
              placeholder="Buscar por cliente, equipo o técnico..."
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
                        <q-icon name="smartphone" color="primary" size="18px" />
                        <span>{{ servicio.equipo }}</span>
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

                  <div class="q-mt-sm text-subtitle1 text-weight-bold">
                    Cobro: ${{ servicio.precio }} 
                    <span class="text-caption text-grey-7">({{ servicio.metodoPago }})</span>
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

                <q-input
                  v-model="formulario.equipo"
                  label="Marca y Modelo del Equipo *"
                  outlined
                  dense
                  placeholder="Ej: iPhone 12, Samsung A15, Xiaomi Redmi Note 10"
                  :rules="[val => (val && val.trim().length > 0) || 'La marca y modelo son obligatorios']"
                />

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

                <q-input
                  v-model="formulario.fechaHora"
                  label="Fecha y Hora de Recepción *"
                  outlined
                  dense
                  placeholder="YYYY-MM-DD HH:mm"
                  :rules="[val => (val && val.trim().length > 0) || 'La fecha y hora son obligatorias']"
                />

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-6">
                    <q-input
                      v-model.number="formulario.precio"
                      type="number"
                      label="Precio Cobrado ($) *"
                      outlined
                      dense
                      :rules="[val => (val !== null && val !== '' && val >= 0) || 'Ingrese un precio válido']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.metodoPago"
                      :options="opcionesMetodoPago"
                      label="Método de Pago *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione un método']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-sm">
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.estadoPago"
                      :options="opcionesEstadoPago"
                      label="Estado del Pago *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione estado del pago']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.estadoEquipo"
                      :options="opcionesEstadoEquipo"
                      label="Estado del Equipo *"
                      outlined
                      dense
                      :rules="[val => !!val || 'Seleccione estado del equipo']"
                    />
                  </div>
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
import { ref } from 'vue'

const CLAVE_STORAGE = 'taller_don_efrain_servicios_v1'

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
  equipo: '',
  tipoReparacion: '',
  tecnico: '',
  fechaHora: '',
  precio: 0,
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

function limpiarFormulario() {
  const ahora = new Date()
  const fechaTexto = ahora.toISOString().slice(0, 10) + ' ' + ahora.toTimeString().slice(0, 5)

  formulario.value = {
    cliente: '',
    equipo: '',
    tipoReparacion: '',
    tecnico: '',
    fechaHora: fechaTexto,
    precio: 0,
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
  formulario.value = JSON.parse(JSON.stringify(servicios.value[index]))
  modalAbierto.value = true
}

function guardarServicio() {
  const datosLimpios = JSON.parse(JSON.stringify(formulario.value))
  datosLimpios.cliente = datosLimpios.cliente ? datosLimpios.cliente.trim() : ''
  datosLimpios.equipo = datosLimpios.equipo ? datosLimpios.equipo.trim() : ''
  
  if (modoEdicion.value) {
    servicios.value[indiceSeleccionado.value] = datosLimpios
  } else {
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
    (servicio.equipo && servicio.equipo.toLowerCase().includes(q)) ||
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
  background-color: #c2b419;
  border-color: #16cf78 !important;
}

.card-pagado {
  background-color: #ffffff;
  border-color: #66bb6a !important;
}

.card-undelivered {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}
</style>
