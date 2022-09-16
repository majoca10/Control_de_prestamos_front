<template>
  <section class="section is-full">
    <div v-show="showEdit" class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">Editar Clientes</h2>

          <Notification v-if="success" type="success" :message="success" />
          <Notification v-if="error" type="danger" :message="error" />

          <form
            v-if="!success"
            method="post"
            @submit.prevent="actualizarClient(idEdit)"
          >
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
                  v-model="apellidosEdit"
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
                  v-model="emailEdit"
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
                  v-model="celEdit"
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
                  v-model="direccionEdit"
                  type="test"
                  class="input"
                  name="direccion"
                  required
                />
              </div>
            </div>
            <div class="control">
              <button type="submit" class="button is-dark is-fullwidth">
                Actualizar cliente
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
    <div class="container is-full">
      <div class="column is-11 is-offset-1">
        <div
          v-show="showTable"
          class="column is-12 is-four-fifths is-offset-one-fifth"
          style="margin-left:0px"
        >
          <div class="colum is-8 is-full">
            <h2 class="title has-text-left">Lista de clientes</h2>
            <button class="button is-small" @click="$router.push('clientsAdd')">
              Agregar
            </button>
          </div>
          <table class="table">
            <thead>
              <tr>
                <th><abbr title="Position">Items</abbr></th>
                <th>Nombre completo</th>
                <th>Documento</th>
                <th>Email</th>
                <th>Cel</th>
                <th>Direccion</th>
                <th>Fecha Creacion</th>
                <th>Comandos</th>
              </tr>
            </thead>
            <tfoot>
              <tr>
                <th><abbr title="Position">Items</abbr></th>
                <th>Nombre completo</th>
                <th>Documento</th>
                <th>Email</th>
                <th>Cel</th>
                <th>Direccion</th>
                <th>Fecha Creacion</th>
                <th>Comandos</th>
              </tr>
            </tfoot>
            <tbody>
              <tr v-for="(user, index) in users" :key="index">
                <th>{{ index + 1 }}</th>
                <td>{{ user.nombres }} {{ user.apellidos }}</td>
                <td>{{ user.documento }}</td>
                <td>{{ user.email }}</td>
                <td>{{ user.cel }}</td>
                <td>{{ user.direccion }}</td>
                <td>{{ formatDate(user.createdAt) }}</td>
                <td style="display: flex;">
                  <button class="button is-small" @click="clientEdit(user.id)">
                    Editar
                  </button>
                  <button
                    class="button is-small"
                    @click="deleteClient(user.id)"
                  >
                    Eliminar
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
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
  async mounted() {
    if (localStorage.getItem('auth._token.local')) {
      const isadmin = await this.$axios.get('users/me')
      console.log(isadmin.data.admin)
      if (isadmin.data.admin === false) {
        this.$router.push('/')
      }
      this.clientList()
    }
  },
  data() {
    return {
      nombres: '',
      apellidos: '',
      documento: '',
      email: '',
      cel: '',
      direccion: '',
      success: null,
      error: null,
      messages: '',
      users: [],
      showTable: true,
      showEdit: false,
      nombresGet: '',
      nombresEdit: '',
      apellidosGet: '',
      apellidosEdit: '',
      documentoGet: '',
      documentoEdit: '',
      emailGet: '',
      emailEdit: '',
      celGet: '',
      celEdit: '',
      cdireccionGet: '',
      direccionEdit: '',
      idGet: '',
      idEdit: ''
    }
  },
  methods: {
    async clientList() {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        const res = await this.$axios
          .get('/clients?_limit=30&_sort=createdAt:desc')
          .then((res) => {
            this.users = res.data
          })
        console.log(res)
        // this.$router.push("clients")
      } catch (e) {
        this.error = e
      }
    },

    async clientEdit(id) {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.get('/clients/' + id).then((res) => {
          this.user = res.data
          this.showEdit = true
          this.showTable = false
          const nombresGet = this.user.nombres
          this.nombresEdit = nombresGet
          const apellidosGet = this.user.apellidos
          this.apellidosEdit = apellidosGet
          const documentoGet = this.user.documento
          this.documentoEdit = documentoGet
          const emailGet = this.user.email
          this.emailEdit = emailGet
          const celGet = this.user.cel
          this.celEdit = celGet
          const direccionGet = this.user.direccion
          this.direccionEdit = direccionGet
          const idGet = this.user.id
          this.idEdit = idGet
        })
      } catch (e) {
        this.error = e
      }
    },

    async actualizarClient(id) {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        const res = await this.$axios.put('/clients/' + id, {
          nombres: this.nombresEdit,
          apellidos: this.apellidosEdit,
          documento: this.documentoEdit,
          email: this.emailEdit,
          cel: this.celEdit,
          direccion: this.direccionEdit
        })
        this.success = 'Cliente actualizado con exito.'
        window.location.reload(true)
        console.log(res.data)
        // this.$router.push("clients")
      } catch (e) {
        this.error = e
      }
    },
    async deleteClient(id) {
      this.error = null
      if (
        window.confirm('Esta seguro que quiere eliminar este Cliente.') === true
      ) {
        try {
          const authorization = 'Authorization'
          this.$axios.defaults.headers.common[
            authorization
          ] = window.localStorage.getItem('auth._token.local')
          const res = await this.$axios.delete('clients/' + id)
          window.location.reload(true)
          console.log(res)
        } catch (e) {
          this.error = e
        }
      } else {
        this.$router.push('/clientsList')
      }
    },
    formatDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('es', options)
    },
    go() {
      this.$router.push('clientsList')
    }
  }
}
</script>
