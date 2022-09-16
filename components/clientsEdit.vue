<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">Editar Clientes</h2>

          <Notification v-if="success" type="success" :message="success" />
          <Notification v-if="error" type="danger" :message="error" />

          <form v-if="!success" method="post" @submit.prevent="editClient">
            <div class="field">
              <label class="label">Nombres</label>
              <div class="control">
                <input
                  v-model="nombresEdit"
                  type="text"
                  class="input"
                  name="nombres"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Apellidos</label>
              <div class="control">
                <input
                  v-model="apellidos"
                  type="text"
                  class="input"
                  name="apellidos"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Email</label>
              <div class="control">
                <input
                  v-model="email"
                  type="email"
                  class="input"
                  name="email"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Celular</label>
              <div class="control">
                <input
                  v-model="cel"
                  type="number"
                  class="input"
                  name="cel"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Direccion</label>
              <div class="control">
                <input
                  v-model="direccion"
                  type="test"
                  class="input"
                  name="direccion"
                  required
                />
              </div>
            </div>
            <div class="control">
              <button type="submit" class="button is-dark is-fullwidth">
                Agregar usuario
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Notification from '~/components/Notification'

export default {
  middleware: 'auth',
  components: {
    Notification
  },
  data() {
    return {
      nombresEdit: '',
      apellidos: '',
      email: '',
      cel: '',
      direccion: '',
      success: null,
      error: null,
      messages: ''
    }
  },
  methods: {
    async editClient() {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.put('clients', {
          nombres: this.nombresEdit,
          apellidos: this.apellidos,
          email: this.email,
          cel: this.cel,
          direccion: this.cel
        })
        this.success = 'Cliente actualizado con exito.'
      } catch (e) {
        this.error = e.response.data.message[0].messages[0].message
      }
    }
  }
}
</script>
