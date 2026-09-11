<template>
  <div class="app">

    <div class="columnas">
      <div class="inicio">
        <h1>Barbería Don Ramíro</h1>
        <p>CORTES EXTREMADAMENTE PREMIUMS</p>
        <button class="botonModal" @click="mostrarModal = true">¿Registrar nuevo servicio?</button>
      </div>
      <div class="informacion">
        <div class="listas">
          <section class="listaSer">
            <div class="tituloLista">
              <h2>SERVICIOS REGISTRADOS</h2>
            </div>
            <div class="contenidoListaSer">
              <p v-if="servicios.length === 0">No hay servicios registrados</p>
              <div class="clientes" v-for="ser in servicios" :key="ser.id" :class="{ confirmadoCard: ser.confirmado }">

                <div class="cardHeader">
                  <h3>{{ ser.cliente }}</h3>
                  <span class="estadoBadge" :class="ser.confirmado ? 'estadoOk' : 'estadoPendiente'">
                    {{ ser.confirmado ? 'Confirmado' : 'Pendiente' }}
                  </span>
                </div>

                <div class="cardBody">
                  <div class="filaInfo">
                    <span class="label">{{ formatearFecha(ser.fecha) }}</span>
                    <span class="valor horaValor">{{ ser.hora }}</span>
                  </div>
                  <div class="servicioDestacado">{{ tServicioTexto(ser.tServicio) }}</div>
                  <div class="filaInfo">
                    <span class="label">Barbero</span>
                    <span class="valor">{{ ser.barbero }}</span>
                  </div>
                </div>

                <div class="filaInfo" v-if="!ser.confirmado">
                  <span class="label">Pago</span>
                  <select class="selectPagoCard" v-model="ser.ePago" @change="actualizarPago(ser)">
                    <option value="Pendiente">Pendiente</option>
                    <option value="Pagado">Pagado</option>
                    <option value="Fiado">Fiado</option>
                  </select>
                </div>

                <div class="filaInfo precioDestacado">
                  <span class="label">Total</span>
                  <span class="valor">{{ formatearPrecio(ser.precio)}}</span>
                </div>

                <div class="cardExtra" v-if="ser.confirmado">
                  <p class="calificacionTexto">{{ '⭐'.repeat(Number(ser.calificacion)) }}</p>
                  <p v-if="ser.observaciones" class="obsTexto">"{{ ser.observaciones }}"</p>
                </div>

                <p class="avisoPago" v-if="!ser.confirmado && ser.ePago !== 'Pagado'">El pago debe estar marcado como "Pagado" para poder confirmar</p>

                <div class="accionesCliente" v-if="!ser.confirmado">
                  <button v-if="!ser.confirmado && ser.ePago === 'Pagado'" class="botonConfirmar" @click="abrirConfirmacion(ser)">✅Confirmar servicioz✅</button>
                  <button class="botonEditar" @click="abrirEdicion(ser)">✏️Editar✏️</button>
                  <button class="botonEliminar" @click="pedirConfirmacion(ser)">❌Eliminar❌</button>
                </div>

              </div>
            </div>
          </section>

          <div class="modal" v-if="clienteAEliminar">
            <div class="modalCont modalConfirmar">
              <h2>¿Estás seguro?</h2>
              <p>Vas a eliminar el registro de <strong>{{ clienteAEliminar.cliente }}</strong></p>
              <div class="bModal">
                <button type="button" class="botonGuardar" @click="confirmarEliminacion">Sí, eliminar</button>
                <button type="button" class="botonCancelar" @click="clienteAEliminar = null">Cancelar</button>
              </div>
            </div>
          </div>

          <div class="modal" v-if="servicioAConfirmar">
            <div class="modalCont modalConfirmarPedido">
              <h2>CONFIRMAR SERVICIO</h2>
              <p>Servicio de <strong>{{ servicioAConfirmar.cliente }}</strong></p>

              <form @submit.prevent="guardarConfirmacion" novalidate>
              <label>Calificación (1 a 5):</label>
              <div class="estrellasSelector">
                <span
                  v-for="n in 5"
                  :key="n"
                  class="estrellaOpcion"
                  :class="{ activa: n <= Number(confirmacionForm.calificacion) }"
                  @click="confirmacionForm.calificacion = n"
                >⭐</span>
              </div>

                <label>Observaciones</label>
                <textarea v-model="confirmacionForm.observaciones"
                  placeholder="Observaciones del servicio..."></textarea>

                <div class="bModal">
                  <button type="submit" class="botonGuardar">Confirmar</button>
                  <button type="button" class="botonCancelar" @click="cancelarConfirmacion">Cancelar</button>
                </div>
              </form>
            </div>
          </div>

          <div class="modal" v-if="clienteAEditar">
            <div class="modalCont">
              <h2>EDITAR SERVICIO</h2>
              <form @submit.prevent="guardarEdicion" novalidate>

                <label>Ingrese el nombre del cliente:</label>
                <input type="text" v-model="formularioEdicion.cliente" placeholder="Ingrese el nombre" required @keydown="bloquearEspacioEdicion">

                <h2>TIPO DE SERVICIO</h2>
                <div class="checkboxes">
                  <div class="opcion">
                    <input type="checkbox" value="cclasico" v-model="formularioEdicion.tServicio" @change="manejarExclusionEdicion('cclasico')">
                    <label>Corte clasico</label>
                  </div>
                  <div class="opcion">
                    <input type="checkbox" value="cmoderno" v-model="formularioEdicion.tServicio" @change="manejarExclusionEdicion('cmoderno')">
                    <label>Corte moderno</label>
                  </div>
                  <div class="opcion">
                    <input type="checkbox" value="barba" v-model="formularioEdicion.tServicio" @change="calcularPrecioEdicion">
                    <label>Barba</label>
                  </div>
                  <div class="opcion">
                    <input type="checkbox" value="cejas" v-model="formularioEdicion.tServicio" @change="calcularPrecioEdicion">
                    <label>Cejas</label>
                  </div>
                  <div class="opcion">
                    <input type="checkbox" value="tinte" v-model="formularioEdicion.tServicio" @change="calcularPrecioEdicion">
                    <label>Tinte</label>
                  </div>
                </div>

                <h2>SELECCIONE UN BARBERO</h2>
                <select v-model="formularioEdicion.barbero" required>
                  <option disabled value="">Seleccione un barbero</option>
                  <option value="Don Ramiro">Don Ramíro</option>
                  <option value="Empleado 1">Empleado 1</option>
                  <option value="Empleado 2">Empleado 2</option>
                </select>

                <h2>FECHA</h2>
                <input type="date" v-model="formularioEdicion.fecha" :min="fechaHoyISO()" required>

                <h2>HORA</h2>
                <input type="time" v-model="formularioEdicion.hora" :min="horaMinimaPara(formularioEdicion.fecha)" required>

                <h2>PRECIO</h2>
                <p>{{ formatearPrecio(formularioEdicion.precio) }}</p>

                <label>Método de pago:</label>
                <select v-model="formularioEdicion.mPago" required>
                  <option disabled value="">Selecciona uno</option>
                  <option value="Efectivo">Efectivo</option>
                  <option value="Transferencia">Transferencia</option>
                  <option value="Tarjeta">Tarjeta</option>
                </select>

                <label>Estado del pago:</label>
                <select v-model="formularioEdicion.ePago" required>
                  <option disabled value="">Selecciona una opción</option>
                  <option value="Pagado">Pagado</option>
                  <option value="Pendiente">Pendiente</option>
                  <option value="Fiado">Fiado</option>
                </select>

                <div class="bModal">
                  <button type="submit" class="botonGuardar">Guardar Cambios</button>
                  <button type="button" class="botonCancelar" @click="cancelarEdicion">Cancelar</button>
                </div>
              </form>
            </div>
          </div>

          <section class="listaSerHoy">
            <div class="tituloLista">
              <h2>SERVICIOS DEL DÍA DE HOY</h2>
            </div>
            <div class="contenidoListaSer">
              <p v-if="serHoy().length === 0">No hay servicios para hoy</p>
              <div class="clientes" v-for="ser in serHoy()" :key="ser.id" :class="{ confirmadoCard: ser.confirmado }">

                <div class="cardHeader">
                  <h3>{{ ser.cliente }}</h3>
                  <span class="estadoBadge" :class="ser.confirmado ? 'estadoOk' : 'estadoPendiente'">
                    {{ ser.confirmado ? 'Confirmado' : 'Pendiente' }}
                  </span>
                </div>

                <div class="cardBody">
                  <div class="filaInfo">
                    <span class="label">{{ formatearFecha(ser.fecha) }}</span>
                    <span class="valor horaValor">{{ ser.hora }}</span>
                  </div>
                  <div class="servicioDestacado">{{ tServicioTexto(ser.tServicio) }}</div>
                  <div class="filaInfo">
                    <span class="label">Barbero</span>
                    <span class="valor">{{ ser.barbero }}</span>
                  </div>
                  <div class="filaInfo">
                    <span class="label">Pago</span>
                    <span class="valor">{{ ser.ePago }}</span>
                  </div>
                </div>

                <div class="filaInfo precioDestacado">
                  <span class="label">Total</span>
                  <span class="valor">{{formatearPrecio(ser.precio)}}</span>
                </div>

                <div class="cardExtra" v-if="ser.confirmado">
                  <p class="calificacionTexto">{{ '⭐'.repeat(Number(ser.calificacion)) }}</p>
                  <p v-if="ser.observaciones" class="obsTexto">"{{ ser.observaciones }}"</p>
                </div>

              </div>
            </div>
          </section>
        </div>
      </div>
    </div>

    <div class="modal" v-if="mostrarModal">
      <div class="modalCont">
        <h2>FORMULARIO DE REGISTRO</h2>
        <form @submit.prevent="guardarForm" novalidate>

          <label>Ingrese el nombre del cliente:</label>
          <input type="text" v-model="formulario.cliente" placeholder="Ingrese el nombre" required @keydown="bloquearEspacio">

          <h2>TIPO DE SERVICIO</h2>
          <div class="checkboxes">
            <div class="opcion">
              <input type="checkbox" id="corteclasico" value="cclasico" v-model="formulario.tServicio"
                @change="manejarExclusion('cclasico')">
              <label>Corte clasico</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="cortemoderno" value="cmoderno" v-model="formulario.tServicio"
                @change="manejarExclusion('cmoderno')">
              <label>Corte moderno</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="barba" value="barba" v-model="formulario.tServicio"
                @change="calcularPrecio">
              <label>Barba</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="cejas" value="cejas" v-model="formulario.tServicio" @change="calcularPrecio">
              <label>Cejas</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="tinte" value="tinte" v-model="formulario.tServicio" @change="calcularPrecio">
              <label>Tinte</label>
            </div>
          </div>

          <h2>SELECCIONE UN BARBERO</h2>
          <select v-model="formulario.barbero" required>
            <option disabled value="">Seleccione un barbero</option>
            <option value="Don Ramiro">Don Ramíro</option>
            <option value="Empleado 1">Empleado 1</option>
            <option value="Empleado 2">Empleado 2</option>
          </select>

          <h2>FECHA</h2>
          <input type="date" v-model="formulario.fecha" :min="fechaHoyISO()" required>

          <h2>HORA</h2>
          <input type="time" v-model="formulario.hora" :min="horaMinimaPara(formulario.fecha)" required>

          <h2>PRECIO</h2>
          <p>{{formatearPrecio(formulario.precio) }}</p>

          <label>Método de pago:</label>
          <select v-model="formulario.mPago" required>
            <option disabled value="">Selecciona uno</option>
            <option value="Efectivo">Efectivo</option>
            <option value="Transferencia">Transferencia</option>
            <option value="Tarjeta">Tarjeta</option>
          </select>

          <label>Estado del pago:</label>
          <select v-model="formulario.ePago" required>
            <option disabled value="">Selecciona una opción</option>
            <option value="Pagado">Pagado</option>
            <option value="Pendiente">Pendiente</option>
            <option value="Fiado">Fiado</option>
          </select>
          <div class="bModal">
            <button type="button" @click="cerrarModal" class="botonCancelar">Cancelar Registro</button>
            <button type="submit" class="botonGuardar">Guardar Registro</button>
          </div>
        </form>
      </div>
    </div>
    <div class="toast" v-if="mensajeAlerta" :class="tipoAlerta">
      {{ mensajeAlerta }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

let servicios = useLocalStorage('ser', [])
let mostrarModal = ref(false)
let mensajeAlerta = ref("")
let tipoAlerta = ref("")

const preciosServicios = {
  cclasico: 25000,
  cmoderno: 30000,
  barba: 15000,
  cejas: 15000,
  tinte: 40000
}

const nombresServicios = {
  cclasico: "Corte clásico",
  cmoderno: "Corte moderno",
  barba: "Barba",
  cejas: "Cejas",
  tinte: "Tinte"
}

function tServicioTexto(tServicio) {
  return tServicio.map(codigo => nombresServicios[codigo]).join(", ")
}

function calcularPrecio() {
  let precio = 0
  for (const servicio of formulario.value.tServicio) {
    precio += preciosServicios[servicio]
  }
  formulario.value.precio = precio
}

let formulario = ref({
  cliente: "",
  tServicio: [],
  barbero: "",
  fecha: "",
  hora: "",
  precio: 0,
  mPago: "",
  ePago: ""
})

function cerrarModal() {
  mostrarModal.value = false
  formulario.value = {
    cliente: "",
    tServicio: [],
    barbero: "",
    fecha: "",
    hora: "",
    precio: 0,
    mPago: "",
    ePago: ""
  }
}

function fechaHoyISO(){
  const hoy = new Date()
  return hoy.getFullYear() + '-' +
    String(hoy.getMonth() + 1).padStart(2, '0') + '-' +
    String(hoy.getDate()).padStart(2, '0')
}

function horaActualStr(){
  const ahora = new Date()
  return String(ahora.getHours()).padStart(2, '0') + ':' + String(ahora.getMinutes()).padStart(2, '0')
}

function horaMinimaPara(fecha){
  if(fecha === fechaHoyISO()){
    return horaActualStr()
  }
  return ''
}

function horaEsValida(fecha, hora){
  if(fecha === fechaHoyISO()){
    return hora >= horaActualStr()
  }
  return true
}

function guardarForm() {
  if(!validarFormulario()){
    return
  }

  servicios.value.push({
    id: Date.now(),
    ...formulario.value,
    confirmado: false,
    calificacion: "",
    observaciones: ""
  })
  cerrarModal()
}

let clienteAEliminar = ref(null)

function pedirConfirmacion(ser) {
  clienteAEliminar.value = ser
}

let servicioAConfirmar = ref(null)
let confirmacionForm = ref({
  calificacion: "",
  observaciones: ""
})

function abrirConfirmacion(ser) {
  if(ser.ePago !== "Pagado"){
    mostrarAlerta("El pago debe estar marcado como Pagado para confirmar el servicio")
    return
  }
  servicioAConfirmar.value = ser
  confirmacionForm.value = {
    calificacion: ser.calificacion || "",
    observaciones: ser.observaciones || ""
  }
}

function cancelarConfirmacion() {
  servicioAConfirmar.value = null
  confirmacionForm.value = { calificacion: "", observaciones: "" }
}

function confirmarEliminacion() {
  const indice = servicios.value.findIndex(s => s.id === clienteAEliminar.value.id)
  servicios.value.splice(indice, 1)
  clienteAEliminar.value = null
}

function fHoy(fs) {
  return fs === fechaHoyISO()
}

function serHoy() {
  return servicios.value.filter(ser => fHoy(ser.fecha))
}

function formatearFecha(fs) {
  if (!fs) return ""
  const [año, mes, dia] = fs.split('-')
  const fecha = new Date(año, mes - 1, dia)
  return fecha.toLocaleDateString('es-CO', {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  })
}

function formatearPrecio(valor){
  return new Intl.NumberFormat('es-CO', {
    style: 'currency',
    currency: 'COP',
    minimumFractionDigits: 0
  }).format(valor)
}

function validarFormulario(){
  if(!formulario.value.cliente.trim()){
    mostrarAlerta("Debes ingresar el nombre del cliente")
    return false
  }
  if(formulario.value.tServicio.length === 0){
    mostrarAlerta("Selecciona al menos un tipo de servicio")
    return false
  }
  if(!formulario.value.barbero){
    mostrarAlerta("Debes seleccionar un barbero")
    return false
  }
  if(!formulario.value.fecha){
    mostrarAlerta("Debes seleccionar una fecha")
    return false
  }
  if(formulario.value.fecha < fechaHoyISO()){
    mostrarAlerta("No puedes agendar una cita en una fecha pasada")
    return false
  }
  if(!formulario.value.hora){
    mostrarAlerta("Debes seleccionar una hora")
    return false
  }
  if(!horaEsValida(formulario.value.fecha, formulario.value.hora)){
    mostrarAlerta("No puedes agendar una hora que ya pasó")
    return false
  }
  if(!formulario.value.mPago){
    mostrarAlerta("Debes seleccionar un método de pago")
    return false
  }
  if(!formulario.value.ePago){
    mostrarAlerta("Debes seleccionar el estado del pago")
    return false
  }
  return true
}

function validarConfirmacion(){
  if(!confirmacionForm.value.calificacion){
    mostrarAlerta("Debes seleccionar una calificación")
    return false
  }
  return true
}

function guardarConfirmacion() {
  if(!validarConfirmacion()){
    return
  }
  if(servicioAConfirmar.value.ePago !== "Pagado"){
    mostrarAlerta("El pago debe estar marcado como Pagado para confirmar el servicio")
    return
  }

  const indice = servicios.value.findIndex(s => s.id === servicioAConfirmar.value.id)
  servicios.value[indice].calificacion = confirmacionForm.value.calificacion
  servicios.value[indice].observaciones = confirmacionForm.value.observaciones
  servicios.value[indice].confirmado = true
  cancelarConfirmacion()
}

function mostrarAlerta(texto, tipo = "error"){
  mensajeAlerta.value = texto
  tipoAlerta.value = tipo
  setTimeout(() => {
    mensajeAlerta.value = ""
  }, 2500)
}

function manejarExclusion(codigoSeleccionado){
  const opuestos = {
    cclasico: "cmoderno",
    cmoderno: "cclasico"
  }

  const opuesto = opuestos[codigoSeleccionado]

  if(opuesto && formulario.value.tServicio.includes(codigoSeleccionado)){
    const indice = formulario.value.tServicio.indexOf(opuesto)
    if(indice !== -1){
      formulario.value.tServicio.splice(indice, 1)
    }
  }

  calcularPrecio()
}

function bloquearEspacio(e){
  if(e.key === " " && formulario.value.cliente.length === 0){
    e.preventDefault()
  }
}

let clienteAEditar = ref(null)
let formularioEdicion = ref({
  cliente: "",
  tServicio: [],
  barbero: "",
  fecha: "",
  hora: "",
  precio: 0,
  mPago: "",
  ePago: ""
})

function abrirEdicion(ser){
  clienteAEditar.value = ser
  formularioEdicion.value = {
    cliente: ser.cliente,
    tServicio: [...ser.tServicio],
    barbero: ser.barbero,
    fecha: ser.fecha,
    hora: ser.hora,
    precio: ser.precio,
    mPago: ser.mPago,
    ePago: ser.ePago
  }
}

function calcularPrecioEdicion(){
  let precio = 0
  for(const servicio of formularioEdicion.value.tServicio){
    precio += preciosServicios[servicio]
  }
  formularioEdicion.value.precio = precio
}

function manejarExclusionEdicion(codigoSeleccionado){
  const opuestos = {
    cclasico: "cmoderno",
    cmoderno: "cclasico"
  }

  const opuesto = opuestos[codigoSeleccionado]

  if(opuesto && formularioEdicion.value.tServicio.includes(codigoSeleccionado)){
    const indice = formularioEdicion.value.tServicio.indexOf(opuesto)
    if(indice !== -1){
      formularioEdicion.value.tServicio.splice(indice, 1)
    }
  }

  calcularPrecioEdicion()
}

function bloquearEspacioEdicion(e){
  if(e.key === " " && formularioEdicion.value.cliente.length === 0){
    e.preventDefault()
  }
}

function validarEdicion(){
  formularioEdicion.value.cliente = formularioEdicion.value.cliente.trimStart()

  if(!formularioEdicion.value.cliente.trim()){
    mostrarAlerta("Debes ingresar el nombre del cliente")
    return false
  }
  if(formularioEdicion.value.tServicio.length === 0){
    mostrarAlerta("Selecciona al menos un tipo de servicio")
    return false
  }
  if(!formularioEdicion.value.barbero){
    mostrarAlerta("Debes seleccionar un barbero")
    return false
  }
  if(!formularioEdicion.value.fecha){
    mostrarAlerta("Debes seleccionar una fecha")
    return false
  }
  if(formularioEdicion.value.fecha < fechaHoyISO()){
    mostrarAlerta("No puedes agendar una cita en una fecha pasada")
    return false
  }
  if(!formularioEdicion.value.hora){
    mostrarAlerta("Debes seleccionar una hora")
    return false
  }
  if(!horaEsValida(formularioEdicion.value.fecha, formularioEdicion.value.hora)){
    mostrarAlerta("No puedes agendar una hora que ya pasó")
    return false
  }
  if(!formularioEdicion.value.mPago){
    mostrarAlerta("Debes seleccionar un método de pago")
    return false
  }
  if(!formularioEdicion.value.ePago){
    mostrarAlerta("Debes seleccionar el estado del pago")
    return false
  }
  return true
}

function guardarEdicion(){
  if(!validarEdicion()){
    return
  }

  const indice = servicios.value.findIndex(s => s.id === clienteAEditar.value.id)
  servicios.value[indice] = {
    ...servicios.value[indice],
    ...formularioEdicion.value
  }

  cancelarEdicion()
  mostrarAlerta("Servicio actualizado correctamente", "exito")
}

function cancelarEdicion(){
  clienteAEditar.value = null
}

function actualizarPago(ser){
  const indice = servicios.value.findIndex(s => s.id === ser.id)
  servicios.value[indice].ePago = ser.ePago
  mostrarAlerta("Estado de pago actualizado a " + ser.ePago, "exito")
}
</script>

<style>
:root {
  --color-principal: #D4AF37;
  --color-secundario: #1E1E1E;
  --color-terciario: #2A2A2A;
}

* {
  padding: 0;
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  box-sizing: border-box;
}

.columnas {
  display: grid;
  grid-template-columns: 350px 1fr;
}

.inicio {
  text-align: center;
  color: var(--color-principal);
  grid-column: 1;
  min-height: 100vh;
  background-color: var(--color-secundario);
  border-right: 1px solid var(--color-principal);
}

.inicio p {
  color: white;
}

.inicio h1 {
  border-bottom: 1px solid var(--color-terciario)
}

.informacion {
  grid-column: 2;
  background-color: rgb(34, 34, 34);
}

.app {
  color: white;
}

.botonModal {
  position: fixed;
  bottom: 0px;
  left: 10.875rem;
  transform: translateX(-50%);
  padding: 1rem 3rem;
  background-color: var(--color-principal);
  border: none;
  border-radius: 10px;
  color: black;
  font-weight: bold;
  cursor: pointer;
  width: max-content;
  font-size: 1rem;
  margin-bottom: 1rem;
}

.botonModal:hover {
  transform: translateX(-50%) scale(1.05);
  transition: all 0.3s;
}

.modal .bModal {
  text-align: center;
  margin-top: 1rem;
}

.modal .botonGuardar {
  padding: 1rem 2rem;
  background-color: var(--color-principal);
  border-radius: 1rem;
  border: none;
  font-weight: bold;
  cursor: pointer;
  background-color: rgb(0, 190, 0);
  color: white;
}

.modal .botonCancelar {
  padding: 1rem 2rem;
  margin-right: 10px;
  background-color: var(--color-principal);
  border-radius: 1rem;
  border: none;
  font-weight: bold;
  cursor: pointer;
  background-color: rgb(170, 2, 2);
  color: white;
}

.modal .botonCancelar:hover {
  scale: 1.05;
  transition: all 0.3s;
}

.modal .botonGuardar:hover {
  scale: 1.05;
  transition: all 0.3s;
}

.listas {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin: 1rem;
}

.tituloLista {
  color: var(--color-principal);
  border-bottom: 2px solid var(--color-terciario);
}

.listaSer {
  grid-column: 1;
  text-align: center;
}

.contenidoListaSer .clientes {
  background-color: var(--color-terciario);
  border-radius: 8px;
  margin-bottom: 0.6rem;
  padding: 0.7rem 0.9rem;
  text-align: left;
}

.cardHeader {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.4rem;
  padding-bottom: 0.35rem;
  border-bottom: 1px solid #3a3a3a;
}

.cardHeader h3 {
  color: var(--color-principal);
  font-size: 1rem;
}

.estadoBadge {
  font-size: 0.65rem;
  font-weight: bold;
  padding: 0.2rem 0.5rem;
  border-radius: 20px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.estadoOk {
  background-color: rgba(46, 204, 113, 0.15);
  color: #2ecc71;
}

.estadoPendiente {
  background-color: rgba(212, 175, 55, 0.15);
  color: var(--color-principal);
}

.cardBody {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.filaInfo {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
}

.filaInfo .label {
  color: #999;
}

.filaInfo .valor {
  color: white;
  text-align: right;
}

.horaValor {
  color: var(--color-principal);
  font-weight: 600;
}

.servicioDestacado {
  color: white;
  font-weight: 600;
  font-size: 0.95rem;
  padding: 0.3rem 0;
}

.selectPagoCard {
  background-color: #1a1a1a;
  color: white;
  border: 1px solid #444;
  border-radius: 6px;
  padding: 0.15rem 0.4rem;
  font-size: 0.8rem;
  cursor: pointer;
}

.selectPagoCard:focus {
  outline: none;
  border-color: var(--color-principal);
}

.precioDestacado {
  margin-top: 0.4rem;
  padding-top: 0.4rem;
  border-top: 1px dashed #3a3a3a;
}

.precioDestacado .valor {
  color: var(--color-principal);
  font-weight: bold;
  font-size: 1.15rem;
}

.cardExtra {
  margin-top: 0.5rem;
  padding-top: 0.5rem;
  border-top: 1px dashed #3a3a3a;
}

.calificacionTexto {
  font-size: 0.85rem;
}

.obsTexto {
  color: #bbb;
  font-size: 0.8rem;
  font-style: italic;
  margin-top: 0.25rem;
}

.avisoPago {
  color: #e74c3c;
  font-size: 0.75rem;
  margin-top: 0.4rem;
}

.accionesCliente {
  display: flex;
  justify-content: flex-end;
  gap: 0.4rem;
  margin-top: 0.6rem;
  flex-wrap: wrap;
}

.botonConfirmar {
  flex: 0 0 auto;
  padding: 0.4rem 0.9rem;
  background-color: #2ecc71;
  color: white;
  border: none;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.8rem;
}

.botonConfirmar:hover {
  filter: brightness(1.1);
}

.botonEditar {
  padding: 0.4rem 0.7rem;
  background-color: transparent;
  color: #3498db;
  border: 1px solid #3498db;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.8rem;
}

.botonEditar:hover {
  background-color: rgba(52, 152, 219, 0.15);
}

.botonEliminar {
  padding: 0.4rem 0.7rem;
  background-color: transparent;
  color: #e74c3c;
  border: 1px solid #e74c3c;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.8rem;
}

.botonEliminar:hover {
  background-color: rgba(231, 76, 60, 0.15);
}

.listaSerHoy {
  grid-column: 2;
  text-align: center;
}

.listaSerHoy h2, .listaSer h2{
  margin-bottom: 1rem;
}

form {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  animation: aparecerFondo 0.5s ease;
  color: black;
}

.modalCont {
  background-color: #1a1a1a;
  color: white;
  padding: 2rem;
  border-radius: 12px;
  border: 2px solid var(--color-principal);
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 0 25px rgba(255, 215, 0, 0.3);
  animation: aparecerModal 0.35s ease;
}

.modalCont h2 {
  color: var(--color-principal);
  text-align: center;
  margin-bottom: 0.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid rgba(255, 215, 0, 0.3);
  letter-spacing: 1px;
}

.modalCont input[type="text"],
.modalCont input[type="date"],
.modalCont input[type="time"],
.modalCont select,
.modalCont textarea {
  background-color: #2a2a2a;
  color: white;
  border: 1px solid #444;
  border-radius: 6px;
  padding: 0.6rem;
  font-size: 1rem;
}

.modalCont input:focus,
.modalCont select:focus,
.modalCont textarea:focus {
  outline: none;
  border-color: var(--color-principal);
}

.modalCont label {
  color: #ccc;
  font-size: 0.9rem;
  margin-top: 0.3rem;
}

.opcion {
  background-color: var(--color-terciario);
  padding: 0.5rem 0.8rem;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  border: 1px solid #444;
}

.opcion input[type="checkbox"] {
  accent-color: var(--color-principal);
  width: 16px;
  height: 16px;
}

.modalCont>form>p {
  background-color: rgba(255, 215, 0, 0.1);
  color: var(--color-principal);
  font-size: 1.3rem;
  font-weight: bold;
  text-align: center;
  padding: 0.5rem;
  border-radius: 6px;
}

.modalConfirmar {
  max-width: 350px;
  text-align: center;
}

.modalConfirmar h2 {
  color: #ff5555;
  border-bottom: none;
}

.checkboxes {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.toast{
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 1rem 1.5rem;
  border-radius: 8px;
  font-weight: bold;
  z-index: 999;
  animation: aparecerToast 0.3s ease;
}

.toast.error{
  background-color: #e74c3c;
  color: white;
}

.toast.exito{
  background-color: #2ecc71;
  color: white;
}

.estrellasSelector {
  display: flex;
  gap: 0.4rem;
  font-size: 1.8rem;
  margin: 0.3rem 0;
}

.estrellaOpcion {
  cursor: pointer;
  filter: grayscale(1);
  opacity: 0.35;
  transition: all 0.15s ease;
}

.estrellaOpcion:hover {
  opacity: 0.8;
  transform: scale(1.1);
}

.estrellaOpcion.activa {
  filter: grayscale(0);
  opacity: 1;
}

.modalCont input[type="date"],
.modalCont input[type="time"] {
  color-scheme: dark;
}

@keyframes aparecerToast {
  from{ opacity: 0; transform: translateX(-50%) translateY(-20px); }
  to{ opacity: 1; transform: translateX(-50%) translateY(0); }
}

@keyframes aparecerModal {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-20px);
  }

  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@media (max-width: 600px) {

  .columnas{
    display: flex;
    flex-direction: column;
  }

  .inicio{
    min-height: auto;
    padding: 1.5rem 1rem;
    border-right: none;
    border-bottom: 1px solid var(--color-principal);
  }

  .botonModal{
    position: static;
    transform: none;
    margin-top: 1rem;
  }

  .botonModal:hover{
    transform: scale(1.05);
  }

  .listas{
    grid-template-columns: 1fr;
    margin: 0.75rem;
    gap: 1.5rem;
  }

  .listaSer,
  .listaSerHoy{
    grid-column: 1;
  }

  .modalCont{
    padding: 1.2rem;
    width: 95%;
  }

  .bModal{
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .modal .botonGuardar,
  .modal .botonCancelar{
    margin-right: 0;
    width: 100%;
  }

}

@media (max-width: 340px) {

  .cardHeader{
    flex-direction: column;
    align-items: flex-start;
    gap: 0.4rem;
  }

  .filaInfo{
    flex-direction: column;
    align-items: flex-start;
    gap: 0.1rem;
  }

  .filaInfo .valor{
    text-align: left;
  }

  .accionesCliente{
    flex-direction: column;
  }

  .checkboxes{
    flex-direction: column;
  }

  .opcion{
    width: 100%;
  }

  .modalCont h2{
    font-size: 1.1rem;
  }

}
</style>