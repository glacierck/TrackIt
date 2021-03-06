<template>
  <div class="login-page window-height window-width bg-light column items-center no-wrap">
    <a v-if="!$q.platform.is.mobile" href="https://github.com/flespi-software/TrackIt/" target="_blank"><img style="position: absolute; top: 0; right: 0; border: 0; width: 149px; height: 149px;" src="../statics/right-graphite@2x.png" alt="Fork me on GitHub"></a>
    <div class="login-back flex items-center justify-center">
      <div class="login-code flex items-center justify-center">
        Track it!
      </div>
    </div>
    <div v-if="!$route.params.token">
      <div class="login-card shadow-4 bg-white column items-center justify-center no-wrap">
        <p class="text-center">Track your devices on the map.</p>
        <div class="row full-width">
          <div class="col-12 text-center">
            <iframe style="width: 100%; height: 96px" :src="`${$flespiServer}/frame/index.html#fff;666;40`" frameborder="0"></iframe>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <div class="login-card shadow-4 bg-white column items-center justify-center no-wrap">
        <q-progress indeterminate color="positive" style="width: 100%; height: 45px" />
      </div>
    </div>
  </div>
</template>

<script>
import { mapMutations } from 'vuex'

export default {
  data () {
    return {
      token: '',
      offlineIntervalId: 0,
      icons: {
        facebook: 'mdi-facebook-box',
        google: 'mdi-google-plus-box',
        live: 'mdi-windows',
        github: 'mdi-github-box',
        email: 'mdi-at'
      }
    }
  },
  computed: {
    model: {
      get () {
        return this.token
      },
      set (val) {
        this.token = val
      }
    }
  },
  methods: {
    ...mapMutations(['setToken']),
    logIn () {
      this.setToken(this.token)
      this.$nextTick(() => { this.$router.push('/') })
    },
    autoLogin () {
      this.setToken(this.$route.params.token)
      setTimeout(() => {
        this.$router.push('/')
      }, 1000)
    },
    checkHasToken () {
      let authCookie = this.$q.cookies.get('X-Flespi-Token'),
        localStorageToken = this.$q.localStorage.get.item('X-Flespi-Token')
      if (this.$route.params && this.$route.params.token) {
        this.autoLogin()
      } else if (localStorageToken) {
        this.token = localStorageToken
        this.logIn()
      } else if (authCookie) {
        this.$q.dialog({
          title: 'Confirm',
          message: `Do you want log in by token ${authCookie}.`,
          ok: true,
          cancel: true
        }).then(() => {
          this.token = authCookie
          this.logIn()
        })
          .catch(() => {})
      }
    }
  },
  watch: {
    $route (val) {
      if (val.params && val.params.token) {
        this.autoLogin()
      }
    }
  },
  created () {
    let tokenHandler = (event) => {
      if (typeof event.data === 'string' && ~event.data.indexOf('FlespiToken')) {
        this.token = event.data
        this.logIn()
        window.removeEventListener('message', tokenHandler)
      }
    }
    window.addEventListener('message', tokenHandler)
    this.$q.loading.hide()
    this.checkHasToken()
  }
}
</script>

<style lang="stylus">
  .row__wrapper
    height 80px
  .login-page
    .login-back
      width 100%
      height 50vh
      overflow hidden
      font-size 8vmax
      background-image url(../statics/mountain.svg)
      background-position center 100px
      background-size contain
      background-repeat no-repeat
      background-color #333
      color rgba(255,255,255,0.9)
      .login-code
        height 50vh
        width: 80vw;
        max-width: 600px;
        background-image url(../statics/trackit.png)
        background-position center
        background-size contain
        background-repeat no-repeat
        opacity .8
        padding-top 20vh
        font-size 80%
    .login-card
      border-radius 2px
      margin-top -50px
      width 80vw
      max-width 600px
      padding 25px
      > i
        font-size 5rem
</style>
