<script>
  import StyledButton from './StyledButton.vue'
  export default {
    components: {
      StyledButton
    },
    props: {
      order: Object,
      isMockup: Boolean
    },
    data() {
      return {
        firstName: '',
        lastName: '',
        phoneClient: '+569',
        socials: '',
        receives: this.order.receives,
        phoneReceives: this.order.phoneReceives,
        takeout: this.order.takeout,
        address: this.order.address,
        addressComment: this.order.addressComment,
        toggleLetter: this.order.letter ? true : false,
        letter: this.order.letter,
        submitFail: false
      }
    },
    computed: {
      takeoutBool() {
        return this.takeout === 'true'
      }
    },
    beforeMount() {
      if (this.order.client) {
        this.phoneClient = this.order.client.phone
        this.socials = this.order.client.socials
        if (this.order.client.name) {
          const arr = this.order.client.name.split(',')
          this.firstName = arr[1].trim()
          this.lastName = arr[0].trim()
        }
      }
    },
    emits: ['ok'],
    methods: {
      formatDate(date) {
        const auxDate = new Date(date)
        if(auxDate.getHours() === 1) {
          return auxDate.toLocaleDateString('es-ES', { 'dateStyle': 'full' })
        }
        return auxDate.toLocaleString('es-ES', {
          'dateStyle': 'full',
          'timeStyle': 'short'
        })
      },
      capitalizeStr(str) {
        return str.split(' ')
          .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
          .join(' ')
      },

      handleSubmit() {
        this.submitFail = false
        const url = `${import.meta.env.VITE_FORM_WORKER}/api/handle-form`
        const body = {
          id: this.order._id,
          fullname: this.capitalizeStr(`${this.lastName}, ${this.firstName}`),
          phone: this.phoneClient,
          socials: this.socials,
          takeout: this.takeout === 'true',
          receives: this.receives,
          phoneReceives: this.phoneReceives,
          address: this.address,
          addressComment: this.addressComment,
          letter: this.letter
        }
        if (this.isMockup) {
          body.dbname = 'dalila-mockup'
        }
        fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(body)
        }).then(response => {
          if (response.status === 200) {
            this.$emit('ok')
          } else {
            this.submitFail = true
          }
        }).catch(err => this.submitFail = true)
      }
    }
  }
</script>

<template>
  <div class="order-info">
    <div class="order-row">
      <h4>Su pedido:</h4>
      <h4>{{order.title}}</h4>
    </div>
    <div class="order-row">
      <h4 class="form-label">Fecha envío:</h4>
      <h4 style="text-align:end;">{{this.formatDate(order.date_delivery)}}</h4>
    </div>
    <p class="text-warning" v-if="new Date(order.date_delivery).getHours() !== 1">
      La hora estipulada arriba es solo una aproximación.
    </p>
    <div class="order-row">
      <h4>Precio:</h4>
      <h4>{{order.price}}</h4>
    </div>
    <div class="order-row">
      <h4>Costo de envío:</h4>
      <h4>{{order.fees}}</h4>
    </div>
    <div class="order-row">
      <h4>Total:</h4>
      <h4 style="font-weight: bold;">{{order.price + order.fees}}</h4>
    </div>
  </div>
  <form class="form-top">
    <div class="form-row">
      <p class="form-label">Nombre de quién envía (opcional)</p>
      <input v-model="firstName"/>
    </div>
    <div class="form-row">
      <p class="form-label">Apellido de quién envía (opcional)</p>
      <input v-model="lastName"/>
    </div>
    <div class="form-row">
      <p class="form-label">Teléfono cliente</p>
      <input v-model="phoneClient"/>
    </div>
    <div class="form-row">
      <p class="form-label">¿Por qué vía nos comunicamos?</p>
      <select v-model="socials">
        <option disabled value="">Elija una...</option>
        <option value="whatsapp">WhatsApp</option>
        <option value="facebook">Facebook</option>
        <option value="instagram">Instagram</option>
        <option value="marketplace">Marketplace</option>
      </select>
    </div>
    <div class="form-row" style="flex-direction: row;">
      <input type="radio" id="retira" value="true" v-model="takeout"/>
      <label for="retira">Retira en florería</label>
      <input type="radio" id="delivery" value="false" v-model="takeout"/>
      <label for="delivery">Delivery</label>
    </div>
    <div class="form-row" v-if="!takeoutBool">
      <p class="form-label">Recibe (nombre y apellido)</p>
      <input v-model="receives"/>
    </div>
    <div class="form-row" v-if="!takeoutBool">
      <p class="form-label">Dirección, Comuna</p>
      <input v-model="address"/>
    </div>
    <div class="form-row" v-if="!takeoutBool">
      <p class="form-label">Referencia de ubicación</p>
      <textarea v-model="addressComment"/>
    </div>
    <div class="form-row" v-if="!takeoutBool">
      <p class="form-label">Teléfono de quien recibe</p>
      <input v-model="phoneReceives" placeholder="+569"/>
    </div>
    <div class="form-row" style="flex-direction: row;">
      <input type="checkbox" id="tarjeta" v-model="toggleLetter" />
      <label for="tarjeta">Lleva tarjeta</label>
    </div>
    <div class="form-row" v-if="toggleLetter">
      <p class="form-label">Contenido de la tarjeta</p>
      <textarea maxlength="120" v-model="letter"/>
    </div>
    <StyledButton class="form-btn" @click="handleSubmit">Enviar</StyledButton>
    <p class="text-warning" v-if="submitFail">
      No pudimos procesar su orden esta vez. Por favor intente de nuevo.
    </p>
  </form>
</template>

<style scoped>
.form-label {
  white-space: nowrap;
  padding-right: 0.5rem;
}
.order-info {
  display: flex;
  flex-direction: column;
  background-color: aliceblue;
  border-radius: 15px;
  padding: 0.5rem 1.5rem;
}
.order-row {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.form-top {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.form-row {
  width: 30vw;
  min-width: 250px;
  display:flex;
  flex-direction: column;
  margin-block: 0.2rem;
}
.form-btn {
  max-width: 150px;
  min-width: 20vw;
  margin-top: 2rem;
  background-color: blueviolet;
  color: white;
  font-weight: 600;
}
.text-warning {
  color: red;
  font-weight: 200;
  font-size: smaller;
}
label {
  margin-inline: 0.5rem;
}
</style>