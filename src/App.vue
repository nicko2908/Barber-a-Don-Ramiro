<template>
  <div class="app">
    <h1>💈Barbería Don Ramíro💈</h1>
    <p>Bienvenido viejón</p>
    <button @click="mostrarModal = true">¿Registrar nuevo servicio?</button>


    <section class="listaSer"> <!--EN ESTA PARTE IRÁN LOS SERVICIOS-->
      <p v-if="servicios.length === 0">No hay servicios registrados</p>

      <div class="tarjeta" v-for="ser in servicios" :key="ser.id">
        <h3>{{ ser.cliente }}</h3>

      </div>
    </section>

    <!--MODALLLLLL MODALLLLL-->
    <div class="modal" v-if="mostrarModal">
      <div class="modalCont"></div>
        <h2>Nuevo servicio viejón</h2>

        <form @submit.prevent="guardarForm">

          <label>cliente</label>
          <input type="text" v-model="formulario.cliente" required>

          <label>Tipo de servicio</label>
          <select v-model="formulario.tServicio" required>
            <option disabled value="">Seleccione uno viejiturro</option>
            <option value="Corte clásico">Corte Clásico</option>
            <option value="Corte moderno">Corte Moderno</option>
            <option value="Barba">Barba</option>
            <option value="Corte + barba">Corte + Barba</option>
            <option value="Cejas">Cejas</option>
            <option value="Tinte">Tinte</option>
          </select>

          <label>Barbero:</label>
          <select v-model="formulario.barbero" required>
            <option disabled value="">Selecciona uno viejito chacarrón</option>
            <option value="Don Ramiro">Viejito Chacarrón</option>
            <option value="Empleado 1">El Brayan</option>
            <option value="Empleado 2">El Kevin</option>
          </select>

          <label>Fecha y Hora:</label>
          <input type="datetime-local" v-model="formulario.fecha" required>

          <label>Precio:</label>
          <input type="number" v-model="formulario.precio" required min="0">

          <label>Método de pago:</label>
          <select v-model="formulario.mPago" required>
            <option disabled value="">Selecciona uno</option>
            <option value="Efectivo">Efectivo</option>
            <option value="Transferencia">Transferencia</option>
            <option value="Tarjeta">Tarjeta</option>
          </select>

          <label>Estado del pago:</label>
          <select v-model="formulario.ePago" required>
            <option disabled value=""></option>
          </select>
          <div class="bModal">
            <button type="submit">Guardar registro viejiturro</button>
            <button type="button" @click="cerrarModal">Nos echamos pa atra'</button>
          </div>
        </form>
    </div>

  </div>
</template>

<script setup>
import {ref} from 'vue'
import {useLocalStorage} from '@vueuse/core'

let servicios = useLocalStorage('ser', [])
let mostrarModal = ref(false)

let formulario = ref({
  cliente:"",
  tServicio:"",
  barbero:"",
  fecha:"",
  precio:"",
  mPago:"",
  ePago:"",
  calificacion:"",
  observaciones:""
})

function cerrarModal(){
  mostrarModal.value=false
  formulario.value = {
  cliente:"",
  tServicio:"",
  barbero:"",
  fecha:"",
  precio:"",
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
</script>

<style>
*{
  padding: 0;
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}
</style>