<script>
  import FormTop from './components/FormTop.vue';
  export default {
    components: {
      FormTop
    },
    data() {
      return {
        order: null,
        loading: true,
        success: false,
        isMockup: false
      }
    },
    beforeMount() {
      const params = new URL(window.location).searchParams
      const orderId = params.get('order-id')
      const mockup = params.get('mockup')
      if (mockup && mockup === 'true') {
        this.isMockup = true
      }
      if (orderId) {
        const mockupSuffix = this.isMockup ? `&mockup=true` : ''
        fetch(`${import.meta.env.VITE_FORM_WORKER}/api/form?id=${orderId}${mockupSuffix}`)
        .then(response => response.json())
        .then(response => {
          this.order = response.document
          return fetch(`${import.meta.env.VITE_FORM_WORKER}/api/client?id=${this.order.clientId}${mockupSuffix}`)
        }).then(response => response.json())
        .then(response => {
          this.order.client = response.document
          this.loading = false
        }).catch(err => this.loading = false)
      } else {
        this.loading = false
      }
    }
  }
</script>

<template>
  <header>
    <div class="wrapper">
      <h3>Floreria Dalila</h3>
    </div>
  </header>

  <main>
    <div v-if="success">
      <h3  style="text-align: center;">
        Para agendar su reserva, realice una transferencia por el monto pactado y enviar comprobante.
      </h3>
      <br/>
      <h3 style="text-align: end;">Cuenta corriente: 
        <b style="font-weight:bolder">XXXXXXXXXX</b>, Banco Itaú</h3>
      <h3 style="text-align: end;">Marcelo Stöckle</h3>
      <h3 style="text-align: end;">Rut: XXXXXXXX-X</h3>
      <h3 style="text-align: end;">Correo: laughingstockperson@gmail.com</h3>
    </div>
    <h3 v-else-if="loading">Cargando...</h3>
    <FormTop
      :order="order"
      :isMockup="isMockup"
      @ok="success=true"
      v-else-if="order"
    />
    <h3 v-else>No se podido procesar su operación.</h3>
  </main>
</template>

<style scoped>
header {
  margin-bottom: 2rem;
}
.wrapper h3 {
  text-align: center;
}
</style>