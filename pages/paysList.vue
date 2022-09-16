<template>
  <section class="section is-full">
    <div class="container is-full">
      <Notification v-if="success" type="success" :message="success" />
      <Notification v-if="error" type="danger" :message="error" />
      <div class="column is-11 is-offset-1">
        <div
          v-show="showTable"
          class="column is-12 is-four-fifths is-offset-one-fifth"
          style="margin-left:0px"
        >
          <div class="colum is-8 is-full">
            <h2 class="title has-text-left">Lista de Pagos</h2>
            <button class="button is-small" @click="$router.push('addpays')">
              Agregar
            </button>
          </div>
          <table class="table">
            <thead>
              <tr>
                <th><abbr title="Position">Items</abbr></th>
                <th>Nombre completo</th>
                <th>Documento</th>
                <th>Monto</th>
                <th>Interes</th>
                <th>Numero de cuotas</th>
                <th>Valor Cuota</th>
                <th>Fecha inicial de pagos</th>
                <th>Comandos</th>
              </tr>
            </thead>
            <tfoot>
              <tr>
                <th><abbr title="Position">Items</abbr></th>
                <th>Nombre completo</th>
                <th>Documento</th>
                <th>Monto</th>
                <th>Interes</th>
                <th>Numero de cuotas</th>
                <th>Valor Cuota</th>
                <th>Fecha inicial de pagos</th>
                <th>Comandos</th>
              </tr>
            </tfoot>
            <tbody>
              <tr v-for="(prestamo, index) in prestamos" :key="index">
                <th>{{ index + 1 }}</th>
                <td>
                  {{ prestamo.client.nombres }}
                  {{ prestamo.client.apellidos }}
                </td>
                <td>
                  {{ prestamo.client.documento }}
                </td>
                <td>
                  {{ prestamo.monto }}
                </td>
                <td>
                  {{ prestamo.intereses }}
                </td>
                <td style="text-align: center;">
                  {{ prestamo.ncuotas }}
                </td>
                <td>
                  {{ prestamo.vcuotas }}
                </td>
                <td>
                  {{ prestamo.fechaipago }}
                </td>
                <td style="display: flex;">
                  <button
                    class="button is-small"
                    @click="deleteprestamo(prestamo.id)"
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
      this.prestamosList()
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
      idEdit: '',
      prestamos: [],
      montoEdit: 0,
      montoGet: 0,
      ncuotasEdit: 0,
      ncuotasGet: 0,
      interesesGet: 0,
      interesesEdit: 0,
      fechaipagoGet: '',
      fechaipagoEdit: '',
      clientnombreEdit: '',
      clientapellidoEdit: '',
      // eslint-disable-next-line
      ppagos: [{'dias': 1, 'name':'Diario'  },{ 'dias': 7, 'name':'Semanal' },{ 'dias': 15, 'name':'Quincenal' },{ 'dias': 30, 'name':'Mensual' }],
      ppagoEdit: '',
      fechasdepagoEdits: [],
      verfecha: false,
      newfipago: '',
      index: 0,
      fechainputnew: ''
    }
  },
  methods: {
    async prestamosList() {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        const res = await this.$axios
          .get('/prestamos?_limit=30&_sort=createdAt:desc&eliminado=false')
          .then((res) => {
            this.prestamos = res.data
          })
        console.log(this.prestamos, res)
        // this.$router.push("clients")
      } catch (e) {
        this.error = e
      }
    },

    async prestamosEdit(id) {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.get('/prestamos/' + id).then((res) => {
          this.prestamosedit = res.data
          this.showEdit = true
          this.showTable = false
          const montoGet = this.prestamosedit.monto
          this.montoEdit = parseInt(montoGet)
          const interesesGet = this.prestamosedit.intereses
          this.interesesEdit = parseInt(interesesGet)
          const ncuotasGet = this.prestamosedit.ncuotas
          this.ncuotasEdit = ncuotasGet
          const fechaipago = this.prestamosedit.fechaipago
          this.fechaipagoEdit = fechaipago
          const nombresclient = res.data.client.nombres
          this.clientnombreEdit = nombresclient
          const apellidosclient = res.data.client.apellidos
          this.clientapellidoEdit = apellidosclient
          const ppagoGet = res.data.ppagos
          if (ppagoGet === 1) {
            this.ppagoEdit = 'Diario'
          }
          if (ppagoGet === 7) {
            this.ppagoEdit = 'Semanal'
          }
          if (ppagoGet === 15) {
            this.ppagoEdit = 'Quincenal'
          }
          if (ppagoGet === 30) {
            this.ppagoEdit = 'Mensual'
          }
          const fechasdepagosGet = this.prestamosedit.fechaspagos
          this.fechasdepagoEdits = fechasdepagosGet
          console.log(res.data)
        })
      } catch (e) {
        this.error = 'hola'
      }
    },
    async deleteprestamo(id) {
      this.error = null
      if (
        window.confirm('Esta seguro que quiere eliminar este Cliente.') === true
      ) {
        try {
          const authorization = 'Authorization'
          this.$axios.defaults.headers.common[
            authorization
          ] = window.localStorage.getItem('auth._token.local')
          const res = await this.$axios.put('prestamos/' + id, {
            eliminado: true
          })
          window.location.reload(true)
          console.log(res)
        } catch (e) {
          this.error = e
        }
      } else {
        this.$router.push('/leansList')
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
