<template>
    <q-layout class="bkg-auth">

        <div class="layout-view">
            <div class="h-100 flex justify-center">
                <div class="row wrap justify-center items-center h-100 content-auth">
                    <form @submit.prevent="entrar">
                        <div style="min-width: 300px">
                          <div>
                            <q-field icon="email">
                              <q-input v-model="login.email" float-label="Email" autocapitalize="none" tabindex="1"/>
                            </q-field>
                          </div>

                          <div>
                            <q-field icon="lock">
                              <q-input type="password" v-model="login.pass" float-label="Senha" autocapitalize="none" tabindex="2"/>
                            </q-field>
                          </div>

                          <div style="padding-top: 20px">
                            <q-field>
                              <q-btn type="submit" tabindex="3" class="full-width" color="brand">
                                Entrar
                              </q-btn>
                            </q-field>

                            <q-field class="text-center">
                              <q-spinner name="oval" color="#9a6aa9" :size="15" v-if="loading"></q-spinner>
                            </q-field>
                          </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>

    </q-layout>

</template>

<script>
  import $ from 'jquery'

  import {
    QLayout,
    QInput,
    QField,
    QBtn,
    QSpinner} from 'quasar'

  export default {
    components: {
      QLayout,
      QInput,
      QField,
      QBtn,
      QSpinner
    },
    data () {
      return {
        pageName: this.$route.name,
        news: [],
        opened: {},
        login: {
          email: 'miklos.pluhar@boxnet.com.br',
          pass: 'Box##0017'
        },
        loading: false
      }
    },
    methods: {

      entrar () {
        this.loading = true
        var data = this.login

        if (window.location.host.indexOf('localhost') < 0) {
          var settings = {
            "async": true,
            "crossDomain": true,
            "url": "http://portal.nxcd.com.br/authenticate/",
            "method": "POST",
            "headers": {
              "content-type": "application/x-www-form-urlencoded",
              "cache-control": "no-cache"
            },
            "data": this.login
          }

          $.ajax(settings).done((response) => {
            localStorage.setItem('token', response.token)
            this.$router.push('/cnh')
          })
        }
        else {
          this.$router.push('/cnh')
        }
      }

    }
  }
</script>

<style lang="stylus">
    @import '~variables'

    .bg-brand
      background-color tomato

    .h-100
        height 100vh !important

    .bkg-auth
        background-color #ededed

        .layout-header,
        .layout-footer
            display none



</style>
