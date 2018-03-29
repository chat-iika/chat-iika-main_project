<template>
  <div>
    <div class="chatbox">
      <input type="text" v-model="message" @keyup.enter="sendChat" />
      <button @click="sendChat">Kirim</button>
    </div>
    <div>
      <ul class="chats">
        <li v-for="(item,index) in chats" :key="index + index">{{ item }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Chat',
  data () {
    return {
      message: '',
      chats: [
        'Iika: Hai, aku Iika. Silakan tanyakan apa saja kepadaku semua hal yang berhubungan dengan pilkada ^o^',
        'Iika: Kalau seandainya Iika ada salah, kasih tau Iika ya :D'
      ],
      prevMessage: '',
      correction: false,
      id: 0
    }
  },
  methods: {
    sendChat () {
      let app = this
      if (this.message === '') return

      if (this.correction) {
        if (this.message.toLowerCase() === 't') {
          axios.post('https://pilkadangu-predictor.herokuapp.com/add/mislabels', {
            message: this.prevMessage
          })
            .then(res => console.log('Sukses'))
            .catch(err => console.log(err))
        }

        this.correction = false
        this.chats.push('Iika: Okay, terimakasih kaka :D')
        this.message = ''
        return
      }

      this.chats.push(`Kaka: ${this.message}`)
      this.chats.push('Iika: ...')

      axios.post('https://pilkadangu-predictor.herokuapp.com/predict', {
        message: app.message
      }).then(res => {
        const intent = res.data.intent
        let response = ''

        switch (intent) {
          case 'greet':
            response = 'Hai juga kaka ^o^ Ada yang bisa Iika bantu?'
            break
          case 'candidate':
            response = 'Cagub cawagub yang Iika tau baru Ridwan Kamil ka, maaf ya :( Tapi tar Iika usahain cari lebih lagi ya ^o^ Anyway jangan lupa pilih Ridwan Kamil ya hehe'
            break
          case 'cadidate-detail':
            response = 'Apa kaka ingin mendapatkan informasi tentang suatu calon? Maaf tapi untuk saat ini Iika belum bisa mengetahui subjek yang kaka cari :('
            break
          case 'party':
            response = 'Iika cuma tau PDIP ka, maaf ya.. :( Tar Iika cari tau lagi hehe'
            break
          case 'time':
            response = 'Kayaknya tanggal 28 Juni 2018 deh kak pilkadanya, tar Iika konfirmasi lagi deh ^o^'
            break
          case 'appreciation':
            response = 'Sama-sama kakak senang membantu kaka ^o^'
            break
          default:
            response = 'Iika ga ngerti maksud kaka gimana .-. Tapi coba tanya Iika yang berhubungan dengan Pilkada pasti Iika coba bantu jawab ^o^'
        }

        app.chats.pop()
        app.chats.push(`Iika: ${response}`)
        app.chats.push('Iika: Apakah Iika menjawab dengan benar ._.? Jawab dengan Y atau T saja kaka ^o^')
        app.correction = true
        app.prevMessage = app.message
        app.message = ''
      }).catch(err => console.log(err))
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  text-align: left;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
