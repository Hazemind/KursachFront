<template>
  <div id="app">
    <div v-if="!pageState">
      <h2>Войдите в аккаунт</h2>
      <input type="text" placeholder="Логин" v-model="login">
      <br>
      <input type="password" placeholder="Пароль" v-model="password">
      <br>
      <b-button v-on:click="checkUser" variant="primary" v-bind:disabled="false">Войти</b-button>
    </div>
    <div v-if="pageState">
      <b-button v-on:click="exit" variant="danger" v-bind:disabled="false" style="height:38px;margin:57px">Выход</b-button>
      <RedactableTable/>
    </div>
    <div v-if="!pageState">
      <OnlyForViewTable/>
    </div>
  </div>
</template>

<script>
import RedactableTable from './components/RedactableTable.vue'
import OnlyForViewTable from "@/components/OnlyForViewTable.vue";
import axios from "axios";

export default {
  name: 'App',
  components: {
    RedactableTable,
    OnlyForViewTable
  },
  data(){
    return{
      pageState:false,
      login:'',
      password:''
    }
  },
  methods:{
    checkUser(){
      axios.get('http://95.216.8.101:8080/login/api/v1/login',{auth: {username: this.login, password: this.password}}).catch(err=>this.checkResponse(err))
          .then(response=>(this.checkResponse(response)))
    },
    checkResponse(res){
      console.log(res)
      if (res.status==200){
        this.pageState = true
      }else{
        alert('Неверный логин или пароль')
      }
    },
    exit(){
      this.pageState = false
      this.login = ''
      this.password = ''
    }
  }
}
</script>

<style scoped>
@import'~bootstrap/dist/css/bootstrap.css';

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin:50px;
  background: lightgray;
}
</style>
