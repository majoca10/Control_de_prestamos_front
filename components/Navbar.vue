<template>
  <nav class="navbar" role="navigation" aria-label="main navigation">
    <div class="navbar-brand">
      <nuxt-link class="navbar-item" to="/">Inicio</nuxt-link>

      <a
        role="button"
        class="navbar-burger burger"
        aria-label="menu"
        aria-expanded="false"
        data-target="navbarBasicExample"
      >
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
      </a>
    </div>

    <div id="navbarBasicExample" class="navbar-menu">
      <div v-if="isAuthenticated" class="navbar-start">
        <div class="navbar-item has-dropdown is-hoverable">
          <a class="navbar-link">
            {{ loggedInUser.username }}
          </a>
          <div class="navbar-dropdown">
            <a class="navbar-item" href="/profile">Perfil</a>
            <hr class="navbar-divider" />
            <a class="navbar-item" href="/clientslist">Clientes</a>
            <hr class="navbar-divider" />
            <a class="navbar-item" href="/leansList">Registrar Prestamo</a>
            <hr class="navbar-divider" />
            <a class="navbar-item" href="/paysList">Registrar Pago</a>
            <hr class="navbar-divider" />
            <a class="navbar-item" @click="logout">Salir</a>
          </div>
        </div>
      </div>

      <div v-if="!isAuthenticated" class="navbar-end">
        <div class="navbar-item">
          <div class="buttons">
            <nuxt-link class="button is-primary" to="/register">
              <strong>Registrar</strong>
            </nuxt-link>
            <nuxt-link class="button is-light" to="/login">
              Ingresar
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['isAuthenticated', 'loggedInUser'])
  },
  methods: {
    async logout() {
      await this.$auth.logout()
    }
  }
}
</script>
