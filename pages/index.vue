<template>
  <div class="container">
    <div v-for="(talk, key, index) in talks" :key=index>
      <p>{{talk}}</p>
    </div>
    <input v-model="content">
    <button v-on:click="inputTalks">add</button>
    <button v-on:click="getMail">GetMail</button>
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
  data: function() {
    return {
      user: {},
      task: {
        'recurrence': [
          'RRULE:FREQ=DAILY;COUNT=1'
        ],
        'attendees': [
        ],
        'reminders': {
          'useDefault': true,
        }
      },
      content: "",
      talks: []
    }
  },
  mounted: function() {
    console.log(this.$gapi)

    this.$gapi.signIn()
      .then(user => {
        console.log('Signed in as %s', user.name)
        console.log(user)
        this.user = user
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
  methods: {
    checkInputs: function() {
      if (this.content.match(/予定の追加/)){
        this.talks.push("予定の名前を入力してください")
        console.log(this.task)
      }
      if (this.content.match(/予定の名前：/)) {
        let title = this.content.replace(/予定の名前：/, '')
        this.task.summary = title
        this.talks.push("予定の開始時間を入力してください")
        console.log(this.task)
      }
      if (this.content.match(/予定の開始時間：/)) {
        let start = this.content.replace(/予定の開始時間：/, '')
        this.task.start = {dateTime: start, timeZone: "Asia/Tokyo"}
        this.talks.push("予定の終了時間を入力してください")
        console.log(this.task)
      }
      if (this.content.match(/予定の終了時間：/)) {
        let end = this.content.replace(/予定の終了時間：/, '')
        this.task.end = {dateTime: end, timeZone: "Asia/Tokyo"}
        this.talks.push("予定を確定しますか？")
        console.log(this.task)
      }
      if (this.content.match(/確定/)) {
        this.$gapi.request({
          path: 'https://www.googleapis.com/calendar/v3/calendars/primary/events',
          method: 'POST',
          body : JSON.stringify(this.task)
        }).then(response => {
          console.log(response)
        })
      }

    },
    inputTalks: function() {
      this.talks.push(this.content);
      this.checkInputs();
      this.content = "";
      this.$forceUpdate();
    },
    getMail: function() {
      this.$gapi.request({
        path: `https://www.googleapis.com/gmail/v1/users/${this.user.id}/messages`,
        method: 'GET'
      }).then((response) => {
        console.log(response)
      })
    }
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
