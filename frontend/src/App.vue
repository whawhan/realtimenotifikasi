<template>
  <div class="container">
    <div class="flex justify-between">
      <div>
        <h1>Send Notification</h1>
        <p>
          Me <input type="text" v-model="me" />
        </p>
        <p>
          To <input type="text" v-model="to" />
        </p>
        <p>
          Message <input type="text" v-model="message" />
        </p>
        <button @click="privateMessage">Send Message Private</button>
        <button @click="sendMessage">Send Message All</button>
        <button @click="addChat">Save Message</button>

        <p>
          <ul>
            <li v-for="(msg, i) in incomingMessages" :key="i">
              {{ msg.name }}
              {{ msg.message }}
            </li>
          </ul>
          <ul>
            <li v-for="(msg, i) in incomingPrivateMessages" :key="i">
              {{ msg.name }}
              {{ msg.message }}
            </li>
          </ul>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { Notyf } from 'notyf';
import 'notyf/notyf.min.css';

export default {

  data: () => ({
    notifier: new Notyf(),
    me: 'person1',
    to: 'person2',
    message: 'Hello',
    incomingMessages: [],
    incomingPrivateMessages: [],
    connection: null,
  }),

  mounted() {
    this.registerWS()
  },

  methods: {
    sendMessage() {
      const data = JSON.stringify({
        to: this.to,
        message: this.message,
      })

      this.connection.send(data)
    },

    privateMessage() {
      const data = JSON.stringify({
        private: true,
        from: this.me,
        to: this.to,
        message: this.message,
      })

      const payload = {
        name: this.me,
        date: (new Date).toISOString(),
        message: this.message
      }

      this.incomingPrivateMessages.unshift(payload)

      this.connection.send(data)
    },

    registerWS() {
      const url = 'ws://localhost:8000'
      this.connection = new WebSocket(url)

      this.connection.onopen = event => {
        console.log('We are connected', event)
      }

      this.connection.onerror = error => {
        console.log(`WebSocket error: ${error}`)
      }

      this.connection.onmessage = event => {
        const data = JSON.parse(event.data)

        // console.log(data)

        if (data.private && data.to === this.me) {
          const payload = {
            name: data.from,
            date: (new Date).toISOString(),
            message: data.message
          }

          this.incomingPrivateMessages.unshift(payload)

          this.notifier.success(data.message);
        } else if (!data.private) {
          const payload = {
            name: data.from,
            date: (new Date).toISOString(),
            message: data.message
          }

          this.incomingMessages.unshift(payload)

          this.notifier.success(data.message);
        }
      }
    },

    async addChat() {
            const backend='http://localhost:4001/chat/'
            const payload = {
                me: this.me,
                to: this.to,
                message: this.message,
            }
            
            const response = await axios.post(backend+'addChat', payload)            
            this.items.push({
                me: this.me,
                to: this.to,
                message: this.message,
            })

            this.me = ''
            this.yo = ''
            this.message = ''
            // this.getList()

        },
  },











  
}
</script>

<style scoped>
.container {
  margin: auto;
  width: 800px;
}
.flex {
    display: flex;
}
.justify-between {
    justify-content: space-between;
}
</style>
