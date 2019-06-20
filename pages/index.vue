<template>
  <div class="container">
  </div>
</template>

<script>
import Vue from 'vue';
import VueGoogleApi from 'vue-google-api'

const config = {
  apiKey: process.env.API_KEY,
  clientId: process.env.CLIENT_ID,
  scope: process.env.SCOPES,
  discoveryDocs: [ process.env.DISCOVERY_DOCS ]
}
Vue.use(VueGoogleApi, config)

export default {
  mounted: function() {
    console.log(this.$gapi)

    this.$gapi.signIn()
      .then(user => {
        console.log('Signed in as %s', user.name)
      })
      .catch(err => {
        console.error('Not signed in: %s', err.error)
      })

    this.$gapi.request({
      path: 'https://www.googleapis.com/calendar/v3/calendars/primary/events',
      method: 'GET',
      params: {
        timeMin: (new Date()).toISOString(),
        maxResults: 10
      }
    }).then(response => {
      console.log(response)
    })
  },
  components: {
    Logo
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
