<template>
  <div
    id="q-app"
    :key="content.locale + content.currency + '-' + content.lastContentUpdatedDate"
  >
    <!-- Hack to re-render the app when locale or currency change -->
    <router-view />
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import * as mutationTypes from 'src/store/mutation-types'

import { get, debounce } from 'lodash'

import { getAuthToken } from 'src/utils/auth'
import EventBus from 'src/utils/event-bus'
import stelace from 'src/utils/stelace'

import contentEditingMixin from 'src/mixins/contentEditing'

export default {
  mixins: [
    contentEditingMixin,
  ],
  sockets: {
    // Move these to mixin if needed in other components
    /* eslint-disable camelcase */
    connect () {
      // console.log('Stelace Signal connected')
    },
    authentication (signal_id, authCallback) {
      const publishableKey = process.env.STELACE_PUBLISHABLE_API_KEY
      const authToken = getAuthToken()
      let channels = []
      const organizationId = !this.isNaturalUser ? this.currentUser.id : undefined

      if (organizationId) channels = [...channels, organizationId]

      if (typeof authCallback === 'function' && publishableKey) authCallback({ publishableKey, authToken, channels })

      this.$socket.client.signal_id = signal_id
    },
    signal (data) {
      // default Signal event name
    },
    appUpdate ({ noCache = true } = {}) {
      // refresh page dialog when app is updated to prevent missing features or broken webpack chunks
      this.$store.commit(mutationTypes.SET_APP_UPDATE_AVAILABLE, { noCache })
    }
    /* eslint-enable camelcase */
  },
  data () {
    return {}
  },
  computed: {
    ...mapGetters([
      'isNaturalUser',
      'currentUser'
    ])
  },
  watch: {
    // if the socket connection has been ended due to too long inactivity
    // recreate the socket
    '$q.appVisible' (visible) {
      if (visible && !this.$socket.connected) {
        this.refreshSocket()
      }
    }
  },
  created () {
    EventBus.$on('error', ({ data, level, notification } = {}) => {
      const defaultMessage = notification === true ? 'error.unknown_happened_header' : ''
      // notification can be an object containing notify options + message
      const message = defaultMessage || get(notification, 'message') || notification

      if (!message) return // show nothing by default

      if (!level || level === 'error') this.notifyFailure(message, notification)
      else this.notify(message)
    })

    this.$router.afterEach(this.handleRouteQuery)
  },
  async mounted () {
    this.$store.dispatch('initApp')

    this.$socket.client.open()

    this.handleUserSessionExpiration()

    // only subscribe in client-side (mounted function)
    EventBus.$on('refreshSocket', () => {
      this.refreshSocket()
    })
  },
  methods: {
    refreshSocket () {
      this.$socket.client.close()
      this.$socket.client.open()
    },
    handleRouteQuery () {
      const hash = this.$route.hash
      // Assuming 2-second delay is enough to prevent infinite reload loop
      // with webpack chunk loading error
      if (hash === '#reload') {
        setTimeout(() => {
          this.$router.replace(Object.assign({}, this.$route, { hash: '' }))
        }, 2000)
      }
    },
    handleUserSessionExpiration () {
      // use debounce function to prevent notification spamming
      // due to multiple requests failing after session expiration
      const debouncedEmitUserSessionExpiredError = debounce(() => {
        this.notifyInfo('error.user_session_expired', { timeout: 10000 })
        const tokenStore = stelace.getTokenStore()
        tokenStore.removeTokens()

        this.$store.dispatch('logout', { sessionExpired: true })
      }, 2000)

      stelace.onError('userSessionExpired', () => {
        debouncedEmitUserSessionExpiredError()
      })
    },
  },
}
</script>

<style></style>
