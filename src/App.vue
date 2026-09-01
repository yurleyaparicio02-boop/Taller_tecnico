<template>
  <q-layout view="lHh Lpr lFf" class="app-shell">
    <q-header class="app-header">
      <q-toolbar class="app-toolbar">
        <div class="brand">
          <div class="brand-name">Taller Don Efraín</div>
          <div class="brand-sub">Servicio técnico de celulares y tablets</div>
        </div>
        <q-space />
        <q-btn
          unelevated
          no-caps
          class="btn-primary"
          icon="add"
          label="Nuevo servicio"
          @click="abrirModalNuevo"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="app-page">

        <div class="stat-strip">
          <div class="stat-cell">
            <div class="stat-value">{{ servicios.length }}</div>
            <div class="stat-label">Servicios totales</div>
          </div>
          <div class="stat-cell">
            <div class="stat-value stat-warn">{{ contarSinEntregar() }}</div>
            <div class="stat-label">Sin entregar</div>
          </div>
          <div class="stat-cell">
            <div class="stat-value stat-danger">{{ contarPendientesPago() }}</div>
            <div class="stat-label">Con pago pendiente</div>
          </div>
          <div class="stat-cell">
            <div class="stat-value stat-accent">${{ totalPorCobrar().toFixed(2) }}</div>
            <div class="stat-label">Total por cobrar</div>
          </div>
        </div>

        <div class="row q-col-gutter-sm q-mb-lg">
          <div class="col-12 col-md-5">
            <q-input
              dense
              outlined
              class="field-clean"
              v-model="busqueda"
              placeholder="Buscar por cliente o equipo"
              prepend-icon="search"
              clearable
            />
          </div>
          <div class="col-xs-12 col-sm-6 col-md-3">
            <q-select
              dense
              outlined
              class="field-clean"
              v-model="filtroEstadoEquipo"
              :options="opcionesEstadoEquipoFiltro"
              label="Estado del equipo"
              emit-value
              map-options
              clearable
            />
          </div>
          <div class="col-xs-12 col-sm-6 col-md-4">
            <q-select
              dense
              outlined
              class="field-clean"
              v-model="filtroEstadoPago"
              :options="opcionesEstadoPagoFiltro"
              label="Estado del pago"
              emit-value
              map-options
              clearable
            />
          </div>
        </div>

        <div class="row q-col-gutter-md">
          <div
            class="col-12 col-sm-6 col-md-4"
            v-for="s in serviciosFiltrados()"
            :key="s.id"
          >
            <q-card flat class="servicio-card">
              <q-card-section class="q-pb-none">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="card-title">{{ s.cliente }}</div>
                    <div class="card-subtitle">{{ s.equipo }}</div>
                  </div>
                  <div class="col-auto status-dot-wrap">
                    <span
                      class="status-dot"
                      :class="'dot-' + s.estadoEquipo"
                      :title="etiquetaEstadoEquipo(s.estadoEquipo)"
                    />
                  </div>
                </div>
              </q-card-section>

              <q-card-section class="q-py-sm">
                <div class="row items-center q-gutter-xs q-mb-sm">
                  <span class="tag-pill"><q-icon name="build" size="13px" /> {{ etiquetaReparacion(s.tipoReparacion) }}</span>
                  <span class="tag-pill"><q-icon name="person" size="13px" /> {{ s.tecnico }}</span>
                </div>

                <div class="meta-line">
                  <q-icon name="event" size="14px" /> {{ s.fecha }} · {{ s.hora }}
                </div>

                <div class="row items-center justify-between q-mt-md">
                  <div class="price-value">${{ Number(s.precio).toFixed(2) }}</div>
                  <span class="pago-pill" :class="'pago-' + s.estadoPago">
                    <q-icon :name="iconoEstadoPago(s.estadoPago)" size="13px" />
                    {{ etiquetaEstadoPago(s.estadoPago) }}
                  </span>
                </div>

                <div v-if="s.estadoPago === 'abono'" class="meta-line q-mt-xs">
                  Abonado ${{ Number(s.montoAbonado || 0).toFixed(2) }} · Saldo
                  <span class="saldo-value">${{ calcularSaldo(s).toFixed(2) }}</span>
                </div>

                <div class="meta-line q-mt-xs">
                  <q-icon name="payments" size="14px" /> {{ etiquetaMetodoPago(s.metodoPago) }}
                </div>

                <div v-if="s.observaciones" class="meta-line meta-note q-mt-sm">
                  <q-icon name="notes" size="14px" /> {{ s.observaciones }}
                </div>

                <div class="row items-center q-mt-sm q-gutter-sm" v-if="s.estadoEquipo === 'entregado'">
                  <q-icon name="star_rate" size="16px" class="text-amber-8" />
                  <q-rating
                    v-model="s.calificacion"
                    max="5"
                    size="20px"
                    color="amber-8"
                    icon="star_border"
                    icon-selected="star"
                    @update:model-value="guardarAutomatico(s)"
                  />
                  <span class="text-caption text-grey-7">{{ s.calificacion > 0 ? s.calificacion + '/5' : 'Calificar' }}</span>
                </div>
                <div class="pending-flag q-mt-sm" v-else>
                  <q-icon name="pending_actions" size="15px" /> Aún sin entregar
                </div>
              </q-card-section>

              <q-separator class="card-divider" />

              <q-card-actions align="between" class="card-actions">
                <div>
                  <q-btn
                    v-if="s.estadoEquipo !== 'entregado'"
                    flat
                    dense
                    no-caps
                    size="sm"
                    class="btn-advance"
                    :label="etiquetaSiguienteEstado(s.estadoEquipo)"
                    @click="avanzarEstado(s)"
                  />
                </div>
                <div>
                  <q-btn flat round dense icon="edit" size="sm" class="icon-btn" @click="abrirModalEditar(s)" />
                  <q-btn flat round dense icon="delete" size="sm" class="icon-btn icon-btn-danger" @click="pedirConfirmacionEliminar(s)" />
                </div>
              </q-card-actions>
            </q-card>
          </div>

          <div class="col-12" v-if="serviciosFiltrados().length === 0">
            <div class="empty-state">
              <q-icon name="inbox" size="40px" />
              <div class="q-mt-sm">No hay servicios que coincidan con la búsqueda.</div>
            </div>
          </div>
        </div>

      </q-page>
    </q-page-container>

    <q-dialog v-model="mostrarModal" persistent>
      <q-card class="modal-card" style="width: 500px; max-width: 95vw;">
        <q-card-section class="modal-header">
          <div class="modal-title">{{ editando ? 'Editar servicio' : 'Nuevo servicio' }}</div>
        </q-card-section>

        <q-form @submit.prevent="guardarServicio">
          <q-card-section style="max-height: 65vh" class="scroll">
            <div class="row q-col-gutter-sm">
              <div class="col-12">
                <q-input
                  outlined dense
                  class="field-clean"
                  v-model="form.cliente"
                  label="Nombre del cliente *"
                  :rules="[val => !!val && val.trim().length > 0 || 'El nombre es obligatorio']"
                />
              </div>
              <div class="col-12">
                <q-input
                  outlined dense
                  class="field-clean"
                  v-model="form.equipo"
                  label="Marca y modelo del equipo *"
                  placeholder="Ej. Samsung A15"
                  :rules="[val => !!val && val.trim().length > 0 || 'El equipo es obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-select
                  outlined dense
                  v-model="form.tipoReparacion"
                  :options="opcionesTipoReparacion"
                  label="Tipo de reparación *"
                  emit-value
                  map-options
                  :rules="[val => !!val || 'Selecciona un tipo']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-select
                  outlined dense
                  v-model="form.tecnico"
                  :options="tecnicos"
                  label="Técnico que atendió *"
                  :rules="[val => !!val || 'Selecciona un técnico']"
                />
              </div>
              <div class="col-xs-12 col-sm-6">
                <q-input
                  outlined dense
                  class="field-clean"
                  type="date"
                  v-model="form.fecha"
                  label="Fecha de recepción *"
                  :rules="[val => !!val || 'Requerida']"
                />
              </div>
              <div class="col-xs-12 col-sm-6">
                <q-input
                  outlined dense
                  class="field-clean"
                  type="time"
                  v-model="form.hora"
                  label="Hora *"
                  :rules="[val => !!val || 'Requerida']"
                />
              </div>
              <div class="col-xs-12 col-sm-6">
                <q-input
                  outlined dense
                  class="field-clean"
                  type="number"
                  step="0.01"
                  v-model.number="form.precio"
                  label="Precio cobrado *"
                  prefix="$"
                  :rules="[val => val !== null && val > 0 || 'Debe ser mayor a 0']"
                />
              </div>
              <div class="col-xs-12 col-sm-6">
                <q-select
                  outlined dense
                  class="field-clean"
                  v-model="form.metodoPago"
                  :options="opcionesMetodoPago"
                  label="Método de pago *"
                  emit-value
                  map-options
                  :rules="[val => !!val || 'Selecciona un método']"
                />
              </div>
              <div class="col-xs-12 col-sm-6">
                <q-select
                  outlined dense
                  class="field-clean"
                  v-model="form.estadoPago"
                  :options="opcionesEstadoPago"
                  label="Estado del pago *"
                  emit-value
                  map-options
                  :rules="[val => !!val || 'Selecciona un estado']"
                />
              </div>
              <div class="col-xs-12 col-sm-6" v-if="form.estadoPago === 'abono'">
                <q-input
                  outlined dense
                  class="field-clean"
                  type="number"
                  step="0.01"
                  v-model.number="form.montoAbonado"
                  label="Monto abonado *"
                  prefix="$"
                  :rules="[val => val !== null && val >= 0 && val <= (form.precio || 0) || 'Debe ser válido y no mayor al precio']"
                />
              </div>
              <div class="col-12">
                <q-select
                  outlined dense
                  class="field-clean"
                  v-model="form.estadoEquipo"
                  :options="opcionesEstadoEquipo"
                  label="Estado del equipo *"
                  emit-value
                  map-options
                  :rules="[val => !!val || 'Selecciona un estado']"
                />
              </div>
              <div class="col-12" v-if="form.estadoEquipo === 'entregado'">
                <div class="row items-center q-mb-md">
                  <q-icon name="star_rate" size="20px" class="text-amber-8 q-mr-sm" />
                  <div class="text-subtitle2 text-weight-bold">Calificación del cliente</div>
                </div>
                <div class="row items-center q-gutter-md">
                  <q-rating
                    v-model="form.calificacion"
                    max="5"
                    size="40px"
                    color="amber-8"
                    icon="star_border"
                    icon-selected="star"
                  />
                  <div class="text-body2">
                    <div v-if="form.calificacion === 0" class="text-grey-7">Haz clic en las estrellas para calificar</div>
                    <div v-else class="text-weight-bold text-amber-8">{{ form.calificacion }} / 5 estrellas ⭐</div>
                  </div>
                </div>
              </div>
              <div class="col-12">
                <q-input
                  outlined dense
                  class="field-clean"
                  type="textarea"
                  v-model="form.observaciones"
                  label="Observaciones (opcional)"
                  autogrow
                />
              </div>
            </div>
          </q-card-section>

          <q-card-actions align="right" class="q-pa-md modal-actions">
            <q-btn flat no-caps label="Cancelar" class="btn-ghost" @click="cerrarModal" />
            <q-btn unelevated no-caps type="submit" label="Guardar" class="btn-primary" />
          </q-card-actions>
        </q-form>
      </q-card>
    </q-dialog>

    <q-dialog v-model="mostrarConfirmEliminar">
      <q-card class="modal-card" style="width: 380px; max-width: 90vw;">
        <q-card-section class="row items-center">
          <q-icon name="warning" size="24px" class="q-mr-sm" style="color: var(--danger)" />
          <span class="modal-title" style="font-size: 1.05rem">¿Eliminar este servicio?</span>
        </q-card-section>
        <q-card-section class="meta-line" v-if="servicioAEliminar">
          Se eliminará el registro de <b>{{ servicioAEliminar.cliente }}</b> ({{ servicioAEliminar.equipo }}). Esta acción no se puede deshacer.
        </q-card-section>
        <q-card-actions align="right" class="modal-actions">
          <q-btn flat no-caps label="Cancelar" class="btn-ghost" @click="mostrarConfirmEliminar = false" />
          <q-btn unelevated no-caps label="Eliminar" class="btn-danger" @click="confirmarEliminar" />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('taller_E', [])
const tecnicos = ['Don Efraín', 'Técnico Carlos', 'Técnico María']

const opcionesTipoReparacion = [
  { label: 'Cambio de pantalla', value: 'pantalla' },
  { label: 'Cambio de batería', value: 'bateria' },
  { label: 'Cambio de pin de carga', value: 'pin_carga' },
  { label: 'Liberación', value: 'liberacion' },
  { label: 'Mantenimiento de software', value: 'software' },
  { label: 'Cambio de flex', value: 'flex' },
  { label: 'Otros', value: 'otros' }
]

const opcionesMetodoPago = [
  { label: 'Efectivo', value: 'efectivo' },
  { label: 'Transferencia', value: 'transferencia' },
  { label: 'Tarjeta', value: 'tarjeta' }
]

const opcionesEstadoPago = [
  { label: 'Pagado', value: 'pagado' },
  { label: 'Pendiente', value: 'pendiente' },
  { label: 'Abono', value: 'abono' }
]

const opcionesEstadoEquipo = [
  { label: 'Recibido', value: 'recibido' },
  { label: 'En reparación', value: 'en_reparacion' },
  { label: 'Listo para entregar', value: 'listo' },
  { label: 'Entregado', value: 'entregado' }
]

const opcionesEstadoEquipoFiltro = opcionesEstadoEquipo
const opcionesEstadoPagoFiltro = opcionesEstadoPago

const busqueda = ref('')
const filtroEstadoEquipo = ref(null)
const filtroEstadoPago = ref(null)

function serviciosFiltrados() {
  return servicios.value
    .filter(s => {
      const texto = (busqueda.value || '').toLowerCase().trim()
      const coincideTexto = !texto ||
        s.cliente.toLowerCase().includes(texto) ||
        s.equipo.toLowerCase().includes(texto)
      const coincideEquipo = !filtroEstadoEquipo.value || s.estadoEquipo === filtroEstadoEquipo.value
      const coincidePago = !filtroEstadoPago.value || s.estadoPago === filtroEstadoPago.value
      return coincideTexto && coincideEquipo && coincidePago
    })
    .slice()
    .sort((a, b) => new Date(b.fecha + 'T' + b.hora) - new Date(a.fecha + 'T' + a.hora))
}

const mostrarModal = ref(false)
const editando = ref(false)

function formularioVacio() {
  const ahora = new Date()
  return {
    id: null,
    cliente: '',
    equipo: '',
    tipoReparacion: null,
    tecnico: null,
    fecha: ahora.toISOString().slice(0, 10),
    hora: ahora.toTimeString().slice(0, 5),
    precio: null,
    metodoPago: null,
    estadoPago: 'pendiente',
    montoAbonado: 0,
    estadoEquipo: 'recibido',
    calificacion: 0,
    observaciones: ''
  }
}

const form = ref(formularioVacio())

function abrirModalNuevo() {
  editando.value = false
  form.value = formularioVacio()
  mostrarModal.value = true
}

function abrirModalEditar(servicio) {
  editando.value = true
  form.value = { ...servicio }
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarAutomatico(servicio) {
  // Fuerza la actualización del localStorage al calificar desde la tarjeta
  servicios.value = [...servicios.value]
}

function guardarServicio() {
  if (form.value.estadoPago !== 'abono') {
    form.value.montoAbonado = 0
  }

  if (editando.value) {
    const idx = servicios.value.findIndex(s => s.id === form.value.id)
    if (idx !== -1) servicios.value[idx] = { ...form.value }
  } else {
    servicios.value.push({
      ...form.value,
      id: Date.now().toString(36) + Math.random().toString(36).slice(2)
    })
  }

  mostrarModal.value = false
}

const mostrarConfirmEliminar = ref(false)
const servicioAEliminar = ref(null)

function pedirConfirmacionEliminar(servicio) {
  servicioAEliminar.value = servicio
  mostrarConfirmEliminar.value = true
}

function confirmarEliminar() {
  servicios.value = servicios.value.filter(s => s.id !== servicioAEliminar.value.id)
  mostrarConfirmEliminar.value = false
}

function avanzarEstado(servicio) {
  const orden = ['recibido', 'en_reparacion', 'listo', 'entregado']
  const idx = orden.indexOf(servicio.estadoEquipo)
  if (idx !== -1 && idx < orden.length - 1) {
    servicio.estadoEquipo = orden[idx + 1]
  }
}

function etiquetaSiguienteEstado(estadoActual) {
  const mapa = {
    recibido: 'Pasar a reparación',
    en_reparacion: 'Marcar como listo',
    listo: 'Marcar como entregado'
  }
  return mapa[estadoActual] || ''
}

function etiquetaReparacion(valor) {
  const encontrado = opcionesTipoReparacion.find(o => o.value === valor)
  return encontrado ? encontrado.label : valor
}

function etiquetaMetodoPago(valor) {
  const encontrado = opcionesMetodoPago.find(o => o.value === valor)
  return encontrado ? encontrado.label : valor
}

function etiquetaEstadoPago(valor) {
  const encontrado = opcionesEstadoPago.find(o => o.value === valor)
  return encontrado ? encontrado.label : valor
}

function etiquetaEstadoEquipo(valor) {
  const encontrado = opcionesEstadoEquipo.find(o => o.value === valor)
  return encontrado ? encontrado.label : valor
}

function iconoEstadoPago(estado) {
  const mapa = { pagado: 'check', pendiente: 'priority_high', abono: 'schedule' }
  return mapa[estado] || 'help'
}

function calcularSaldo(servicio) {
  return Number(servicio.precio || 0) - Number(servicio.montoAbonado || 0)
}

function contarSinEntregar() {
  return servicios.value.filter(s => s.estadoEquipo !== 'entregado').length
}

function contarPendientesPago() {
  return servicios.value.filter(s => s.estadoPago === 'pendiente' || s.estadoPago === 'abono').length
}

function totalPorCobrar() {
  return servicios.value.reduce((total, s) => {
    if (s.estadoPago === 'pendiente') return total + Number(s.precio || 0)
    if (s.estadoPago === 'abono') return total + calcularSaldo(s)
    return total
  }, 0)
}
</script>

<style>
:root {
  --bg: #f7f8f7;
  --surface: #ffffff;
  --ink: #1c2321;
  --ink-soft: #5b6461;
  --ink-faint: #8a918e;
  --border: #e2e5e3;
  --accent: #2b6e63;
  --accent-soft: #e4eeec;
  --danger: #b3261e;
  --danger-soft: #fbeae9;
  --warn: #b4690e;
  --warn-soft: #fdf1e4;
}

.app-shell,
.app-shell .q-field,
.app-shell .q-btn {
  font-family: 'Inter', -apple-system, sans-serif;
}
</style>

<style scoped>

.app-page {
  background: var(--bg);
  min-height: 100vh;
  color: var(--ink);
  padding: 24px 32px 48px;
  max-width: 1200px;
  margin: 0 auto;
}

@media (max-width: 599px) {
  .app-page {
    padding: 16px 12px 32px;
  }
}

.app-header {
  background: var(--surface);
  color: var(--ink);
  border-bottom: 1px solid var(--border);
  box-shadow: none;
}

.app-toolbar {
  padding: 14px 20px;
  flex-wrap: wrap;
  row-gap: 10px;
}

@media (max-width: 599px) {
  .app-toolbar {
    padding: 12px 14px;
  }
  .app-toolbar .btn-primary {
    width: 100%;
  }
}

.brand {
  min-width: 0;
}

.brand-name {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 600;
  font-size: 1.15rem;
  letter-spacing: -0.01em;
  line-height: 1.2;
}

@media (max-width: 599px) {
  .brand-name {
    font-size: 1.02rem;
  }
}

.brand-sub {
  font-size: 0.78rem;
  color: var(--ink-soft);
  margin-top: 1px;
}

@media (max-width: 380px) {
  .brand-sub {
    display: none;
  }
}

.btn-primary {
  background: var(--accent);
  color: #fff;
  font-weight: 500;
  font-size: 0.85rem;
  padding: 0 16px;
  border-radius: 6px;
}

.btn-ghost {
  color: var(--ink-soft);
  font-weight: 500;
  font-size: 0.85rem;
}

.btn-danger {
  background: var(--danger);
  color: #fff;
  font-weight: 500;
  font-size: 0.85rem;
  border-radius: 6px;
}

.btn-advance {
  color: var(--accent);
  font-weight: 500;
  font-size: 0.78rem;
}

.icon-btn {
  color: var(--ink-faint);
}
.icon-btn:hover {
  color: var(--ink-soft);
}
.icon-btn-danger:hover {
  color: var(--danger);
}

.stat-strip {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 8px;
  margin-bottom: 24px;
  overflow: hidden;
}

.stat-cell {
  padding: 16px 18px;
  border-right: 1px solid var(--border);
}
.stat-cell:last-child {
  border-right: none;
}

@media (max-width: 599px) {
  .stat-strip {
    grid-template-columns: repeat(2, 1fr);
  }
  .stat-cell {
    padding: 12px 14px;
    border-bottom: 1px solid var(--border);
  }
  .stat-cell:nth-child(2n) {
    border-right: none;
  }
  .stat-cell:nth-last-child(-n + 2) {
    border-bottom: none;
  }
  .stat-value {
    font-size: 1.3rem;
  }
}

.stat-value {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 600;
  font-size: 1.6rem;
  color: var(--ink);
  line-height: 1.1;
}
.stat-warn { color: var(--warn); }
.stat-danger { color: var(--danger); }
.stat-accent { color: var(--accent); }

.stat-label {
  font-size: 0.75rem;
  color: var(--ink-soft);
  margin-top: 4px;
}

.field-clean :deep(.q-field__control) {
  background: var(--surface);
  border-radius: 6px;
}
.field-clean :deep(.q-field__control):before {
  border-color: var(--border);
}

.servicio-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 10px;
  transition: border-color 0.15s ease;
}
.servicio-card:hover {
  border-color: var(--ink-faint);
}

.card-title {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 600;
  font-size: 1rem;
  color: var(--ink);
}

.card-subtitle {
  font-size: 0.78rem;
  color: var(--ink-soft);
  margin-top: 1px;
}

.status-dot-wrap {
  padding-top: 2px;
}
.status-dot {
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--ink-faint);
}
.dot-recibido { background: var(--ink-faint); }
.dot-en_reparacion { background: var(--warn); }
.dot-listo { background: #2f7fb0; }
.dot-entregado { background: var(--accent); }

.tag-pill {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  background: var(--bg);
  border: 1px solid var(--border);
  color: var(--ink-soft);
  font-size: 0.72rem;
  padding: 3px 8px;
  border-radius: 5px;
}

.meta-line {
  font-size: 0.78rem;
  color: var(--ink-soft);
  display: flex;
  align-items: center;
  gap: 4px;
  flex-wrap: wrap;
}

.meta-note {
  color: var(--ink);
  align-items: flex-start;
}

.price-value {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 600;
  font-size: 1.2rem;
  color: var(--ink);
}

.saldo-value {
  color: var(--danger);
  font-weight: 600;
}

.pago-pill {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.72rem;
  font-weight: 500;
  padding: 4px 9px;
  border-radius: 999px;
}
.pago-pagado { background: var(--accent-soft); color: var(--accent); }
.pago-pendiente { background: var(--danger-soft); color: var(--danger); }
.pago-abono { background: var(--warn-soft); color: var(--warn); }

.pending-flag {
  font-size: 0.78rem;
  font-weight: 500;
  color: var(--warn);
  display: flex;
  align-items: center;
  gap: 4px;
}

.card-divider {
  background: var(--border);
}

.card-actions {
  padding: 8px 12px;
}

.empty-state {
  background: var(--surface);
  border: 1px dashed var(--border);
  border-radius: 10px;
  padding: 48px 24px;
  text-align: center;
  color: var(--ink-faint);
}

.modal-card {
  border-radius: 10px;
}
.modal-header {
  border-bottom: 1px solid var(--border);
}
.modal-title {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 600;
  font-size: 1.1rem;
  color: var(--ink);
}
.modal-actions {
  border-top: 1px solid var(--border);
}
</style>