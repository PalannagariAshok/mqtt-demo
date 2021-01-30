<template>
  <div id="app" style="display:flex;width:100%;height:100%;justify-content:center;margin-top:20%">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <div style="display:flex;flex-direction:column">
    <div style="display:flex;margin-bottom:1.5em">
      <button :class="['button', buttonValue1===1? 'active':'inactive' ]" @click="doPublish('output1')" >Output 1</button>
      <!-- <button :class="['button', color1=='active'? 'active':'inactive' ]" style="margin-left:10px;">Output 2</button> -->
    </div>
   
      
  </div>
    
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import mqtt from 'mqtt';
export default {
  name: 'App',
  components: {
    // HelloWorld
  },
  data(){
    return{
      buttonValue1:0,
      color:'active',
      color1:'',
      connection: {
        host: 'broker.mqttdashboard.com',
        port:8000,
        endpoint: '/mqtt',
        clean: true, // Reserved session
        connectTimeout: 4000, // Time out
        reconnectPeriod: 4000, // Reconnection interval
        // Certification Information
        // clientId: 'mqttjs_3be2c321',
        // username: 'emqx_test',
        // password: 'emqx_test',
      },
      subscription: {
        topic: 'demo/fiver/send',
        qos: 0,
      },
      // publish: {
      //   topic: 'topic/fiver/recieve',
      //   qos: 0,
      //   payload: '',
      // },
      // publication:{
      //   topic: 'topic/fiver/recieve',
      //   qos: 0,
      // },
      receiveNews: '',
      qosList: [
        { label: 0, value: 0 },
        { label: 1, value: 1 },
        { label: 2, value: 2 },
      ],
      client: {
        connected: false,
      },
      subscribeSuccess: false,

    }
  },
  mounted(){
     const { host, port, endpoint, ...options } = this.connection
      const connectUrl = `ws://${host}:${port}${endpoint}`
      try {
        this.client = mqtt.connect(connectUrl, options)
      } catch (error) {
        console.log('mqtt.connect error', error)
      }
      this.client.on('connect', () => {
        console.log('Connection succeeded!')
      })
      this.client.on('error', error => {
        console.log('Connection failed', error)
      })
      this.client.on('message', (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message)
        
        let Message = JSON.parse(message);
        console.log(`Received message ${Message.value} from topic ${topic}`)
        if(topic=='demo/fiver/send'){
        
            this.buttonValue1=Message.value
            console.log(this.buttonValue1);
          
        }
      })


      const {  qos } = this.subscription
      this.client.subscribe('demo/fiver/send', { qos }, (error, res) => {
        if (error) {
          console.log('Subscribe to topics error', error)
          return
        }
        this.subscribeSuccess = true
          console.log('Subscribe to topics res', res)
        })
  },
  methods: {
    // Create connection
    createConnection() {
      // Connect string, and specify the connection method used through protocol
      // ws unencrypted WebSocket connection
      // wss encrypted WebSocket connection
      // mqtt unencrypted TCP connection
      // mqtts encrypted TCP connection
      // wxs WeChat mini app connection
      // alis Alipay mini app connection
     
    },
    doSubscribe() {
      
    },
    doPublish() {
      const {  qos} = this.publication
      let payload={};
      let buttonValue1;
      console.log(this.buttonValue1)
      if(this.buttonValue1 ==0){
        buttonValue1=1;
      }
      if(this.buttonValue1 ==1){
        buttonValue1=0;
      }
      // if(val=='output1'){
        
      
      // if(val=='output2'){
      //   payload={
      //     name:val,
      //     'valve':this.buttonValue1
      //   }
      // }
       payload={
        name:'output1',
        value:buttonValue1
      }
      console.log(payload)
      this.client.publish('demo/fiver/receive', JSON.stringify(payload), qos, error => {
        if (error) {
          console.log('Publish error', error)
        }
      })
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}
.button{
  padding:8px 20px 8px 20px;
  outline:none;
  border:none;
  cursor:pointer;
  border-radius:5px;
  color:white;
}
.active{
  background:green;
}
.inactive{
  background:red;
}
</style>
