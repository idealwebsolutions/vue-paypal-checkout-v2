<template>
  <div id="app">
    <PaypalButton currency="USD" amount="3.00" :sdkParams="sdkParams" @sdk-ready="isSdkReady" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import PaypalButton, {PaypalSDKParams} from './components/PaypalButton.vue';

@Component({
  components: {
    PaypalButton,
  },
})

export default class App extends Vue {
  isSdkReady (ready: boolean) {
    console.log(`is ready: ${ready}`)
  }

  data () {
    return ({
      sdkParams: {
        clientId: process.env.VUE_APP_CLIENT_ID,
        disableFunding: 'credit'
      }
    })
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
  margin-top: 60px;
}
</style>
