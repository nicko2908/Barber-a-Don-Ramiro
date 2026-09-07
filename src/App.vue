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
            <h2>SERVICIOS REGISTRADOS</h2>
            <p v-if="servicios.length === 0">No hay servicios registrados</p>
            <div class="clientes" v-for="ser in servicios" :key="ser.id">
              <h3>{{ ser.cliente }}</h3>
              <p>{{ser.precio}}</p>
              <p>{{ser.fecha}}</p>
              <button @click="pedirConfirmacion(ser)">ELIMINAR</button>
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

          <section class="listaSerHoy">
            <h2>SERVICIOS DEL DÍA DE HOY</h2>
            <p v-if="serHoy().length === 0">No hay servicios para hoy</p>
            <div class="clientes" v-for="ser in serHoy()" :key="ser.id">
              <h3>{{ ser.cliente }}</h3>
              <p>{{ser.precio}}</p>
              <p>{{ser.fecha}}</p>        
            </div>      
          </section>
        </div>
      </div>
    </div>

    <div class="modal" v-if="mostrarModal">
      <div class="modalCont">
        <h2>FORMULARIO DE REGISTRO</h2>
        <form @submit.prevent="guardarForm">

          <label>Ingrese el nombre del cliente:</label>
          <input type="text" v-model="formulario.cliente" placeholder="Ingrese el nombre" required>

          <h2>TIPO DE SERVICIO</h2>
          <div class="checkboxes">
            <div class="opcion">
              <input type="checkbox" id="corteclasico" value="cclasico" v-model="formulario.tServicio" @change="calcularPrecio">
              <label>Corte clasico</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="cortemoderno" value="cmoderno" v-model="formulario.tServicio" @change="calcularPrecio">
              <label>Corte moderno</label>
            </div>
            <div class="opcion">
              <input type="checkbox" id="corteybarba" value="cybarba" v-model="formulario.tServicio" @change="calcularPrecio">
              <label>Corte + barba</label>
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
          <input type="date" v-model="formulario.fecha" required>

          <h2>HORA</h2>
          <input type="time" v-model="formulario.hora" required>

          <h2>PRECIO</h2>
          <p>${{ formulario.precio }}</p>

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
          </select>

          <label>Calificación (1 a 5):</label>
          <select v-model="formulario.calificacion" required>
            <option disabled value="">Selecciona una</option>
            <option value="1">⭐</option>
            <option value="2">⭐⭐</option>
            <option value="3">⭐⭐⭐</option>
            <option value="4">⭐⭐⭐⭐</option>
            <option value="5">⭐⭐⭐⭐⭐</option>
          </select>

          <label>Observaciones</label>
          <textarea v-model="formulario.observaciones"></textarea>
          <div class="bModal">
            <button type="submit" class="botonGuardar">Guardar Registro</button>
            <button type="button" @click="cerrarModal" class="botonCancelar">Cancelar Registro</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref} from 'vue'
import {useLocalStorage} from '@vueuse/core'

let servicios = useLocalStorage('ser', [])
let mostrarModal = ref(false)

const preciosServicios = {
  cclasico: 25000,
  cmoderno: 30000,
  cybarba: 35000, 
  cejas: 15000,
  tinte: 40000
}

function calcularPrecio(){
  let precio = 0
  for(const servicio of formulario.value.tServicio){
    precio += preciosServicios[servicio]
  }
  formulario.value.precio = precio
}


let formulario = ref({
  cliente:"",
  tServicio:[],
  barbero:"",
  fecha:"",
  hora:"",
  precio:0,
  mPago:"",
  ePago:"",
  calificacion:"",
  observaciones:""
})

function cerrarModal(){
  mostrarModal.value = false
  formulario.value = {
    cliente:"",
    tServicio: [],
    barbero:"",
    fecha:"",
    hora:"",
    precio: 0,
    mPago:"",
    ePago:"",
    calificacion:"",
    observaciones:""
  }
}

function guardarForm() {
  servicios.value.push({
    id: Date.now(),
    ...formulario.value
  })
  cerrarModal()
}

let clienteAEliminar = ref(null)

function pedirConfirmacion(ser){
  clienteAEliminar.value = ser
}

function confirmarEliminacion(){
  const indice = servicios.value.findIndex(s => s.id === clienteAEliminar.value.id)
  servicios.value.splice(indice, 1)
  clienteAEliminar.value = null
}

function fHoy(fs){
  const hoy = new Date()
  const hoyStr = hoy.getFullYear() + '-' + 
    String(hoy.getMonth() + 1).padStart(2, '0') + '-' + 
    String(hoy.getDate()).padStart(2, '0')
  return fs === hoyStr
}

function serHoy(){
  return servicios.value.filter(ser => fHoy(ser.fecha))
}
</script>

<style>
*{
  padding: 0;
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  box-sizing: border-box;
}

.columnas{
  display: grid;
  grid-template-columns: 350px 1fr;
}

.inicio{
  text-align: center;
  background-color: rgb(0, 0, 93);
  color: gold;
  grid-column: 1;
  min-height: 100vh;
  background-color: rgb(30, 30, 30);
  border-right: 1px solid gold;
  position: relative;
}

.inicio p{
  color: white;
}

.informacion{
  grid-column: 2;
  background-color: black;
}

.app{
  background-color: rgb(0, 0, 78);
  color: white;
}

.botonModal{
  position: absolute;
  bottom: 0px;
  left: 50%;
  transform: translateX(-50%);
  padding: 1rem 3rem;
  background-color: gold;
  border: none;
  border-radius: 10px;
  color: black;
  font-weight: bold;
  cursor: pointer;
  width: max-content;
  font-size: 1rem;
  margin-bottom: 1rem;
}

.botonModal:hover{
  transform: translateX(-50%) scale(1.05);
  transition: all 0.3s;
}

.modal .bModal{
  text-align: center;
  margin-top: 1rem;
}

.modal .botonGuardar{
  padding: 1rem  2rem;
  margin-right: 10px;
  background-color: gold;
  border-radius: 1rem;
  border: none;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-weight: bold;
  cursor: pointer;
}

.modal .botonCancelar{
  padding: 1rem 2rem;
  background-color: gold;
  border-radius: 1rem;
  border: none;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-weight: bold;
  cursor: pointer;
}

.modal .botonCancelar:hover{
  scale: 1.05;
  transition: all 0.3s;
}

.modal .botonGuardar:hover{
  scale: 1.05;
  transition: all 0.3s;
}

.listas{
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin: 1rem;
}

.listaSer{
  grid-column: 1;
  border: solid 3px white;
}

.listaSerHoy{
  grid-column: 2;
  border: solid 3px white;
}

form{
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.modal{
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

.modalCont{
  background-color: #1a1a1a;
  color: white;
  padding: 2rem;
  border-radius: 12px;
  border: 2px solid gold;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 0 25px rgba(255, 215, 0, 0.3);
  animation: aparecerModal 0.35s ease;
}

.modalCont h2{
  color: gold;
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
.modalCont textarea{
  background-color: #2a2a2a;
  color: white;
  border: 1px solid #444;
  border-radius: 6px;
  padding: 0.6rem;
  font-size: 1rem;
}

.modalCont input:focus,
.modalCont select:focus,
.modalCont textarea:focus{
  outline: none;
  border-color: gold;
}

.modalCont label{
  color: #ccc;
  font-size: 0.9rem;
  margin-top: 0.3rem;
}

.opcion{
  background-color: #2a2a2a;
  padding: 0.5rem 0.8rem;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  border: 1px solid #444;
}

.opcion input[type="checkbox"]{
  accent-color: gold;
  width: 16px;
  height: 16px;
}

.modalCont > form > p{
  background-color: rgba(255, 215, 0, 0.1);
  color: gold;
  font-size: 1.3rem;
  font-weight: bold;
  text-align: center;
  padding: 0.5rem;
  border-radius: 6px;
}

.modalConfirmar{
  max-width: 350px;
  text-align: center;
}

.modalConfirmar h2{
  color: #ff5555;
  border-bottom: none;
}

.checkboxes{
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
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
</style>