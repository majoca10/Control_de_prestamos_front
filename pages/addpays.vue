<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">
            Crear pago
          </h2>

          <Notification v-if="success" type="success" :message="success" />
          <Notification v-if="error" type="danger" :message="error" />

          <form>
            <div class="field">
              <label class="label">Buscar Cliente</label>
              <div style="display:flex">
                <div class="control">
                  <input
                    v-model="documento"
                    type="number"
                    class="input"
                    name="documento"
                    placeholder="Numero de Documento"
                    required
                  />
                </div>
                <div style="margin: 5px auto;" class="control">
                  <button
                    type="button"
                    class="button is-dark is-small"
                    @click="
                      buscaClient(documento)
                      active = 0
                    "
                    :disabled="documento.length < 1"
                    :active="active == 0"
                  >
                    Buscar
                  </button>
                </div>
              </div>
            </div>
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
                  <label class="label">Itereses</label>
                  <div class="control is-3">
                    <CurrencyImputReadonly v-model="intereses" />
                  </div>
                </div>
                <div>
                  <label class="label">V. Cuota</label>
                  <div class="control is-3">
                    <CurrencyImputReadonly v-model="vcuota" />
                  </div>
                </div>
              </div>
            </div>
            <div class="field is-12">
              <div class="is-6" style="display:flex;">
                <div
                  class="is-6"
                  style="position:relative; padding-right: 85px;"
                >
                  <label class="label">Periodo de pago</label>
                  <div class="is-info is-3">
                    <p>{{ ppagosx }}</p>
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
                      readonly
                      placeholder="Numero de Cuotas"
                    />
                  </div>
                </div>
              </div>
            </div>
            <div
              class="control"
              v-for="(fechaspagox, index) in fechaspagosx"
              :key="index"
              style="display: flex;"
            >
              <label class="label">Pago {{ index + 1 }}</label>
              <div class="control" style="margin-right: 5px;">
                {{ $moment(fechaspagox).format('MMMM DD YYYY') }}
              </div>
              <label class="switch" style="margin-inline: auto;">
                <input type="checkbox" v-model="checked.status[index + 1]" />
                <span class="slider round"></span>
              </label>
            </div>
            <div class="control">
              <button
                type="button"
                class="button is-dark is-fullwidth"
                style="margin-top:5px"
                @click="
                  actualizarPrestamo(checked.status)
                  active = 0
                "
              >
                Agregar Pago
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
import Currency from '~/components/Currency'
import CurrencyImputReadonly from '~/components/CurrencyImputReadonly'

export default {
  middleware: 'auth',
  // eslint-disable-next-line
  components: { Notification, Currency, CurrencyImputReadonly },
  data() {
    return {
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
      intereses: 0,
      ppagosx: '',
      fechaspagosx: '',
      checked: {
        status: []
      },
      index: 0
    }
  },
  methods: {
    generarPago(a) {
      console.log(a)
    },
    async actualizarPrestamo() {
      this.error = ''
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.put('prestamos', {
          fechaipago: this.fipago
        })
        this.success = 'Prestamo registrado con exito.'
        window.location.reload(true)
      } catch (e) {
        this.error = e
      }
    },
    async buscaClient(cc) {
      this.error = null
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.get('/clients/documento/' + cc).then((res) => {
          this.user = res.data
          if (res.data.length === 0) {
            this.error = 'No existe este documento'
            this.nombrecompleto = ''
          } else {
            this.nombrecompleto =
              this.user[0].nombres + ' ' + this.user[0].apellidos
            this.buscarprestamo(this.user[0]._id)
            console.log(this.user[0]._id)
          }
        })
        // this.$router.push("clients")
      } catch (e) {
        this.error = e
      }
    },
    calculaPrestamo(monto, interes, selectedpp, ncuotas, fipago) {
      // eslint-disable-next-line
      this.versimulador = true
      const diasdepago = parseInt(ncuotas) * parseInt(selectedpp)
      const interesporcentual = parseInt(interes) / 100
      const interesdiario =
        (parseInt(monto) / 30) * interesporcentual.toFixed(2)
      const interespagar = diasdepago * interesdiario.toFixed(2)
      this.vipagar = Math.ceil(interespagar)
      const montoconinteres = parseInt(monto) + interespagar
      this.totalpagar = Math.ceil(montoconinteres)
      const valorcuota = montoconinteres / parseInt(ncuotas)
      this.vcuota = Math.ceil(valorcuota)
      const diashoras = selectedpp
      if (diashoras === 1) {
        this.fpago = 'Diario'
      }
      if (diashoras === 7) {
        this.fpago = 'Semanal'
      }
      if (diashoras === 15) {
        this.fpago = 'Quincenal'
      }
      if (diashoras === 30) {
        this.fpago = 'Mensual'
      }
      const diaEnMilisegundos = 1000 * 60 * 60 * 24 * selectedpp
      const fechas = []
      this.fechaspagos = fechas
      for (let index = 0; index < ncuotas; index++) {
        // eslint-disable-next-line
        let cambiarfecha = null
        if (fechas.length >= 1) {
          cambiarfecha = new Date(fechas[fechas.length - 1]).getTime()
        } else {
          cambiarfecha = new Date(fipago.split('-')).getTime()
        }
        const newconverfecha = cambiarfecha + diaEnMilisegundos
        fechas.push(new Date(newconverfecha))
      }
      this.vergenerarprestamo = true
      console.log('dias', diasdepago)
      console.log('monto', parseInt(monto))
      console.log('interes', interesporcentual)
      console.log('interes diario', interesdiario)
      console.log('interes a pagar', interespagar)
      console.log('total a pagar', montoconinteres)
      console.log('numero de cuotas', ncuotas)
      console.log('valor de la cuota', valorcuota)
      console.log('date', diashoras)
      console.log(this.fechaspagos)
    },
    async buscarprestamo(id) {
      console.log(id)
      await this.$axios.get('/prestamos/cliente/' + id).then((res) => {
        const prestamo = res.data[0]
        const monto = prestamo.monto
        this.monto = parseInt(monto)
        const intereses = prestamo.intereses
        this.intereses = parseInt(intereses)
        const ppagos = prestamo.ppagos
        if (ppagos === '1') {
          this.ppagosx = 'Diario'
        }
        if (ppagos === '7') {
          this.ppagosx = 'Semanal'
        }
        if (ppagos === '15') {
          this.ppagosx = 'Quincenal'
          console.log('2entro')
        }
        if (ppagos === '30') {
          this.ppagosx = 'Mensual'
        }
        const ncuotas = prestamo.ncuotas
        this.ncuotas = ncuotas
        const fechaspagos = prestamo.fechaspagos
        this.fechaspagosx = fechaspagos
        const vcuota = prestamo.vcuotas
        this.vcuota = parseInt(vcuota)
        console.log(prestamo)
      })
    }
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
