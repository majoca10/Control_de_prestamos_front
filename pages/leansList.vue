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
            <h2 class="title has-text-left">Lista de Prestamos</h2>
            <button class="button is-small" @click="$router.push('addLeans')">
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
                  <button
                    class="button is-small"
                    @click="pagarprestamo(prestamo.id)"
                  >
                    Pagos
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="column is-4 is-offset-4">
        <form v-show="showEdit">
          <div class="field">
            <label class="label">Nombre Completo</label>
            <div class="control">
              <input
                v-model="nombrecompleto"
                type="text"
                class="input"
                name="nombrecompleto"
                required
                readonly
                placeholder="Cliente"
              />
            </div>
          </div>
          <div class="field is-12">
            <div class="is-6" style="display:flex;">
              <div>
                <label class="label">Monto solicitado</label>
                <div class="control is-3">
                  <CurrencyImputReadonly v-model="monto" />
                </div>
              </div>
              <div>
                <label class="label">Interes</label>
                <div class="control is-3">
                  <input
                    v-model="interes"
                    type="number"
                    class="input"
                    name="interes"
                    required
                    placeholder="Interes"
                    readonly
                  />
                </div>
              </div>
            </div>
            <div>
              <label class="label">Valor cuota</label>
              <div class="control is-3">
                <CurrencyImputReadonly v-model="vcuota" />
              </div>
            </div>
          </div>
          <div class="field is-12">
            <div class="is-6" style="display:flex;">
              <div class="is-6" style="position:relative; padding-right: 85px;">
                <label class="label">Periodo de pago</label>
                <div class="control is-3">
                  <input
                    v-model="selectedpp"
                    type="text"
                    class="input"
                    name="ppago"
                    required
                    placeholder="Periodo de pagos"
                    readonly
                  />
                </div>
              </div>
              <div>
                <label class="label">Numero de Cuotas</label>
                <div class="control is-3">
                  <input
                    v-model="ncuotas"
                    type="number"
                    class="input"
                    name="ncuotas"
                    required
                    placeholder="Numero de Cuotas"
                    readonly
                  />
                </div>
              </div>
              <div style="display:none;">
                <label class="label">Id</label>
                <div class="control is-3">
                  <input
                    v-model="id"
                    type="number"
                    class="input"
                    name="ncuotas"
                    required
                    placeholder="Numero de Cuotas"
                    readonly
                  />
                </div>
              </div>
            </div>
          </div>
          <div v-show="showEdit" class="is-full">
            <table class="table">
              <h2 class="label">Fechas de pago</h2>
              <tr v-for="(fechaspago, index) in fechaspagos" :key="index">
                <th>Cuota {{ index + 1 }}</th>
                <th style="font-weight: 100;">
                  {{ $moment(fechaspagos[index].fecha).format('MM DD YYYY') }}
                </th>
                <th style="display:flex">
                  <label class="switch" style="margin-inline: auto;">
                    <input
                      type="checkbox"
                      v-model="fechaspagos[index].status"
                    />
                    <span class="slider round"></span>
                  </label>
                  <div
                    v-show="fechaspagos[index].status"
                    class="file is-small"
                    style="margin-left: 20px;"
                  >
                    <label class="file-label">
                      <input
                        class="file-input"
                        type="file"
                        name="resume"
                        @change="uploapay(fechaspagos[index], index)"
                      />
                      <span class="file-cta">
                        <span class="file-icon">
                          <i class="fas fa-upload"></i>
                        </span>
                        <span class="file-label">
                          File…
                        </span>
                      </span>
                    </label>
                  </div>
                  <a
                    v-show="fechaspagos[index].file"
                    :href="'http://127.0.0.1:1337' + fechaspagos[index].file"
                    target="_blank"
                  >
                    Ver
                  </a>
                </th>
              </tr>
            </table>
          </div>
          <div class="control">
            <button
              type="button"
              class="button is-dark is-fullwidth"
              style="margin-top:5px"
              @click="
                generarPago(id, fechaspagos)
                active = 0
              "
            >
              Agregar Pago
            </button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<script>
/* eslint-disable */
import Notification from '~/components/Notification'
import CurrencyImputReadonly from '~/components/CurrencyImputReadonly'

export default {
  middleware: 'auth',
  components: {
    Notification,
    CurrencyImputReadonly
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
      fechainputnew: '',
      nombres: '',
      apellidos: '',
      documento: '',
      email: '',
      cel: '',
      direccion: '',
      success: '',
      error: '',
      messages: '',
      selectedpp: '',
      // eslint-disable-next-line
      ppagos: [{'dias': 1, 'name':'Diario'  },{ 'dias': 7, 'name':'Semanal' },{ 'dias': 15, 'name':'Quincenal' },{ 'dias': 30, 'name':'Mensual' }],
      monto: 0,
      interes: '',
      nombrecompleto: '',
      fipago: '',
      ncuotas: '',
      versimulador: false,
      vipagar: 0,
      vcuota: 0,
      fechaspagos: [],
      fechaspago: [],
      active: false,
      isInputActive: false,
      price: 0,
      totalpagar: 0,
      fpago: '',
      vergenerarprestamo: false,
      filepago: '',
      checked: {
        status: []
      },
      index: 0,
      id: '',
      prestamo: ''
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

    async generarPago(id, fechaspagos) {
      console.log('mirando el status', fechaspagos)
      this.error = ''
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.put('prestamos/' + id, {
          fechaspagos: this.fechaspagos
        })
        this.success = 'Pago registrado con exito.'
        window.location.reload(true)
      } catch (e) {
        this.error = e
      }
    },

    async pagarprestamo(id) {
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
          this.nombrecompleto =
            // eslint-disable-next-line
            this.prestamosedit.client.nombres
            // eslint-disable-next-line
             +' ' +
            this.prestamosedit.client.apellidos
          this.fechaspagos = this.prestamosedit.fechaspagos
          this.monto = parseInt(this.prestamosedit.monto)
          this.interes = parseInt(this.prestamosedit.intereses)
          this.selectedpp = this.prestamosedit.ppagos
          if (this.selectedpp === 1) {
            this.selectedpp = 'Diario'
          }
          if (this.selectedpp === 7) {
            this.selectedpp = 'Semanal'
          }
          if (this.selectedpp === 15) {
            this.selectedpp = 'Quincenal'
          }
          if (this.selectedpp === 30) {
            this.selectedpp = 'Mensual'
          }
          this.ncuotas = parseInt(this.prestamosedit.ncuotas)
          const cuota = ((this.monto + this.interes) / this.ncuotas)
          this.vcuota = parseInt(cuota)
          this.id = this.prestamosedit._id
          console.log('PRESTAMO', res.data, this.fechaspagos, this.id)
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
    },
    async uploapay(idfile, index){
      console.log('donde esta?', idfile, index)
      const formdata = new FormData();
      Array.from(event.target.files).forEach(image => {
          formdata.append('files', image);
      });
      if(formdata){
          let responsefile = await this.$axios.post('upload', formdata, {
              onUploadProgress: progressEvent => {
                console.log('subida de archivo renta', parseInt(Math.round((progressEvent.loaded / progressEvent.total)* 100)), '%')
                const responsefile = parseInt(Math.round((progressEvent.loaded / progressEvent.total)* 100))
                this.responsefile = responsefile
              }
            })
            this.imgperfil = responsefile.data[0]
            this.url = this.imgperfil.url
            const file = this.url
            const array = this.fechaspagos
            const indexarr = index
            this.updatearr(array, indexarr, file)
            console.log(this.imgperfil.url)
      }
      //delete this.$axios.defaults.headers.common["Authorization"];
  },
  updatearr(array, index, file) {
        array[index].file = file;
  },
  }
}
</script>
<style>
.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: 0.4s;
  transition: 0.4s;
}

.slider:before {
  position: absolute;
  content: '';
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  -webkit-transition: 04s;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #2196f3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196f3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(26px);
  -ms-transform: translateX(26px);
  transform: translateX(26px);
}

/* Rounded sliders */
.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}
</style>
