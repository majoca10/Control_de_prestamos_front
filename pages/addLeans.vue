<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">
            Simulador y generador de prestamo
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
                    <Currency v-model="monto" />
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
                    />
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
                  <div class="select is-info is-3">
                    <select v-model="selectedpp">
                      <option
                        v-for="(ppago, index) in ppagos"
                        :key="index"
                        :value="ppago.dias"
                      >
                        {{ ppago.name }}
                      </option>
                    </select>
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
                    />
                  </div>
                </div>
              </div>
            </div>
            <div class="field">
              <label class="label">Fecha inicial de pago</label>
              <div class="control">
                <input
                  v-model="fipago"
                  type="date"
                  class="input"
                  name="fipago"
                  required
                  min="01-01-2022"
                  max="31-12-2030"
                  step="1"
                  placeholder="Fecha inicial"
                  data-date-format="DD MMMM YYYY"
                />
              </div>
            </div>
            <div v-show="versimulador" class="is-full">
              <table class="table">
                <h2 class="label">Resumen</h2>
                <tr>
                  <th>
                    Monto
                  </th>
                  <th style="font-weight: 100;">
                    <CurrencyImputReadonly v-model="monto" />
                  </th>
                </tr>
                <tr>
                  <th>
                    Interes
                  </th>
                  <th style="font-weight: 100;">
                    <CurrencyImputReadonly v-model="vipagar" />
                  </th>
                </tr>
                <tr>
                  <th>
                    Pagos
                  </th>
                  <th style="font-weight: 100;">
                    {{ fpago || 'Sin seleccion' }}
                  </th>
                </tr>
                <tr>
                  <th>
                    Numero de cuotas
                  </th>
                  <th style="font-weight: 100;">
                    {{ ncuotas || 0 }}
                  </th>
                </tr>
                <tr>
                  <th>
                    Valor Cuota
                  </th>
                  <th style="font-weight: 100;">
                    <CurrencyImputReadonly v-model="vcuota" />
                  </th>
                </tr>
                <tr>
                  <th>
                    Total a pagar
                  </th>
                  <th style="font-weight: 100;">
                    <CurrencyImputReadonly v-model="totalpagar" />
                  </th>
                </tr>
              </table>

              <table class="table">
                <h2 class="label">Fechas de pago</h2>
                <tr v-for="(fechaspago, index) in fechaspagos" :key="index">
                  <th>Cuota {{ index + 1 }}</th>
                  <th style="font-weight: 100;">
                    {{ $moment(fechaspagos[index].fecha).format('MM DD YYYY') }}
                  </th>
                  <th style="font-weight: 100;">
                    Pago
                    <label class="switch" style="margin-inline: auto;">
                      <input
                        type="checkbox"
                        v-model="fechaspagos[index].status"
                      />
                      <span class="slider round"></span>
                    </label>
                  </th>
                  <th style="font-weight: 100;"></th>
                </tr>
              </table>
            </div>
            <div class="control">
              <button
                type="button"
                class="button is-dark is-fullwidth"
                @click="
                  calculaPrestamo(monto, interes, selectedpp, ncuotas, fipago)
                  active = 0
                "
                :disabled="
                  monto.length < 1 ||
                    interes.length < 1 ||
                    selectedpp.length < 1 ||
                    ncuotas.length < 1 ||
                    fipago.length < 1
                "
                :active="active == 0"
              >
                Simular Prestamo
              </button>
              <button
                type="button"
                class="button is-dark is-fullwidth"
                style="margin-top:5px"
                @click="
                  generarPrestamo()
                  active = 0
                "
                :disabled="!vergenerarprestamo"
              >
                Agregar Prestamo
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
      vergenerarprestamo: false
    }
  },
  methods: {
    async generarPrestamo() {
      this.error = ''
      try {
        const authorization = 'Authorization'
        this.$axios.defaults.headers.common[
          authorization
        ] = window.localStorage.getItem('auth._token.local')
        await this.$axios.post('prestamos', {
          client: this.user[0].id,
          fechaipago: this.fipago,
          monto: this.monto.toString(),
          intereses: this.vipagar.toString(),
          totalpagar: this.totalpagar.toString(),
          ncuotas: this.ncuotas,
          vcuotas: this.vcuota.toString(),
          fechaspagos: this.fechaspagos,
          eliminar: false,
          ppagos: this.selectedpp.toString(),
          finalizado: false
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
          console.log(res.data)
          if (res.data.length === 0) {
            this.error = 'No existe este documento'
            this.nombrecompleto = ''
          } else {
            this.nombrecompleto =
              this.user[0].nombres + ' ' + this.user[0].apellidos
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
      const diaEnMilisegundos =
        1000 * 60 * 60 * 24 + 1000 * 60 * 60 * 24 * selectedpp
      const fechas = []
      for (let index = 0; index < ncuotas; index++) {
        // eslint-disable-next-line
        let cambiarfecha = null
        if (fechas.length >= 1) {
          const index = fechas.length
          const position = index - 1
          const fechaarr = fechas[position].fecha
          cambiarfecha = new Date(fechaarr)
        } else {
          // eslint-disable-next-line
          cambiarfecha = new Date(fipago)
          cambiarfecha.setHours(24)
        }
        const newconverfecha = new Date(cambiarfecha)
        const year = newconverfecha.getFullYear()
        const month = newconverfecha.getMonth()
        // eslint-disable-next-line
        const lastday = new Date(year, month +1, 0).getDate()
        if (lastday === 31) {
          // eslint-disable-next-line
          const diadepago = newconverfecha.setDate(
            new Date(cambiarfecha).getDate() + selectedpp + 1
          )
          const newdiapago = new Date(newconverfecha).getDate()
          console.log(newdiapago)
          if (newdiapago === 16) {
            // eslint-disable-next-line
            const diadepago = newconverfecha.setDate(
              new Date(cambiarfecha).getDate() + selectedpp - 1
            )
          }
        } else {
          newconverfecha.setDate(new Date(cambiarfecha).getDate() + selectedpp)
        }
        console.log(selectedpp)
        fechas.push({
          fecha: new Date(newconverfecha),
          status: false,
          file: ''
        })
      }
      this.fechaspagos = fechas
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
      console.log(this.fechaspagos, diaEnMilisegundos)
      console.log('fecha inicial', fipago)
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
