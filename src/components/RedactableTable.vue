<template>
  <div>
    {{login}}
    <!-- Кнопки -->
    <div>
      <b-button variant="primary" v-on:click="addToTable">Добавить запись</b-button>
      <b-button variant="primary" v-on:click="showResponse">Показать записи</b-button>
    </div>
    <!-- Контейнер для таблицы и действий с таблицой -->
    <b-container class="mt-2" style="min-width: 90%;">
      <br>
      <div style="alignment: center; display:flex; justify-content: space-between; max-height: 38px">
        <!--Фильтры таблицы-->
        <div style="alignment: center; display:flex; justify-content: space-between; max-height: 38px">
          <input style="display:inline-block" type="text" placeholder="Фильтр" v-model="tableFilter">
          <b-button v-on:click="clearFilter">X</b-button>
        </div>
        <b-form-checkbox-group
            v-model="filterOn"
            class="mt-2"
            style="height: 2px"
        >
          <b-form-checkbox value="cellNumber">Номер ячейки</b-form-checkbox>
          <b-form-checkbox value="productNumber">Количество товаров</b-form-checkbox>
          <b-form-checkbox value="slotSize">Размер ячейки</b-form-checkbox>
        </b-form-checkbox-group>
        <b-form-select label-field="Название продукта" style="max-width: 13%" :options="productNamesList"  v-model="tableFilter"/>
        <b-datepicker style="max-width: 350px" v-model="tableFilter" range ></b-datepicker>
        <!--Скролл таблицы-->
        <b-pagination
            v-model="currentPage"
            :total-rows="rows"
            :per-page="perPage"
            aria-controls="warehouseTable"
            style="min-width:19.5%"
        ></b-pagination>
      </div>
      <!-- Сама таблица -->
      <b-table class="mt-4" hover id="warehouseTable" :items="warehouse" :fields="warehouseFields" :per-page="perPage"
               :current-page="currentPage" :filter="tableFilter" :filter-included-fields="filterOn" :sort-by="sortBy" small>
        <template v-slot:cell(cellNumber)="row">
          <b-form-input v-model="row.item.cellNumber"/>
        </template>
        <template v-slot:cell(productNumber)="row">
          <b-form-input v-model="row.item.productNumber"/>
        </template>
        <template v-slot:cell(slotSize)="row">
          <b-form-input v-model="row.item.slotSize"/>
        </template>
        <template v-slot:cell(productName)="row">
          <b-form-select v-model="row.item.productName" :options="productNamesList" />
        </template>
        <template v-slot:cell(rowAction)="row">
          <b-button class="delete-button" variant="primary" v-on:click="saveToDatabase(row.item.cellNumber)">Сохранить</b-button>
          <b-button class="delete-button" variant="danger" v-on:click="deleteThisRow(row.item.cellNumber)">Удалить</b-button>
        </template>
      </b-table>
    </b-container>
  </div>
</template>

<script>
import Vue from "vue";
import VueAxios from "vue-axios";
import axios from "axios";

Vue.use(VueAxios, axios)
export default {
  data() {
    return {
      productNamesList:[],
      warehouse: [],
      dbStorageTable:[],
      dbProductTable:[],
      warehouseFields:[
        {key:'cellNumber',label:'Номер ячейки'},
        {key:'productNumber',label:'Количество товаров'},
        {key:'slotSize',label:'Размер ячейки'},
        {key:'productName',label:'Название продукта'},
        {key:'dateOfChange',label:'Дата изменения'},
        {key:'rowAction',label:'Действие'}
      ],
      tableFilter: '',
      currentPage:1,
      perPage:11,
      filterOn:[],
      sortBy: 'rowAction',
      tableReady:false
    }
  },
  props:[
    'login',
    'password'
  ],
  methods:{
    alertSaveResponse(al){ 
      console.log(al.status)
      if (al.status===200){
        alert('Данные успешно загружены')
      }else{
        alert('Проверьте размер ячейки или количество товаров')
      }
    },
    alertDeleteResponse(al){
      console.log(al.status)
      if (al.status===200){
        alert('Данные успешно удалены')
      }
    },
    clearFilter(){
      this.tableFilter =''
    },
    addToTable(){
      let date = new Date().toISOString().slice(0, 19).replace('T', ' ')
      this.warehouse.push({cellNumber:this.warehouse.length+1,dateOfChange:date})
    },
    deleteThisRow(index){
      let tableSize = this.warehouse.length
      let counter = 0
      while (counter < tableSize){
        if (this.warehouse[counter].cellNumber == (index)){
          this.warehouse.splice(counter,1)
          axios.delete('http://95.216.8.101:8080/storage/api/v1/deleteStorage?id='+index,{auth: {username: 'admin', password: 'admin'}}).catch(err=>err)
              .then(response=>{this.alertDeleteResponse(response);this.getLoadStorageList()})
          break
        }
        counter++
      }
    },
    showResponse(){
      if (this.tableReady){
        let length = this.dbStorageTable.data.length
        let counter = 0
        let date = new Date()
        this.warehouse = []
        this.getProductNames()
        while (counter < length) {
          date = this.dbStorageTable.data[counter].lastChangeTime.slice(0, 19).replace('T', ' ');
          this.warehouse.push({
            cellNumber: this.dbStorageTable.data[counter].cellNumber,
            productNumber: this.dbStorageTable.data[counter].quanity,
            slotSize: this.dbStorageTable.data[counter].cellSize,
            productName: this.dbStorageTable.data[counter].productId,
            dateOfChange: date
          })
          counter++
        }
        this.listProductNames()
      }
    },
    listProductNames(){
      let length = this.dbProductTable.data.length
      this.productNamesList=[]
      for (let i=0;i<length;i++){
        this.productNamesList.push(this.dbProductTable.data[i].name)
      }
    },
    getProductNames(){
      let lengthProduct=this.dbProductTable.data.length
      let lengthStorage=this.dbStorageTable.data.length
      let counterP = 0
      let counterS = 0
      while((counterS<lengthStorage)){
        while (counterP<lengthProduct){
          if (this.dbStorageTable.data[counterS].productId == this.dbProductTable.data[counterP].id){
            this.dbStorageTable.data[counterS].productId = this.dbProductTable.data[counterP].name
          }
          counterP++
        }
        counterP=0
        counterS++
      }
    },
    saveToDatabase(index){
      let test
      for(let i =0;i<this.warehouse.length;i++){
        if (index == this.warehouse[i].cellNumber){
          test = {...this.warehouse[i]}
          break
        }
      }
      let lengthProduct=this.dbProductTable.data.length
      for (let i=0;i<lengthProduct;i++){
        if (this.dbProductTable.data[i].name == test.productName){
          test.productName = this.dbProductTable.data[i].id
          delete test['dateOfChange']
          test.quanity = test.productNumber
          test.cellSize = test.slotSize
          test.productId = test.productName
          delete test['productNumber']
          delete test['slotSize']
          delete test['productName']
          axios.post('http://95.216.8.101:8080/storage/api/v1/saveStorage',test, {auth: {username: 'admin', password: 'admin'}}).catch(err=>err)
              .then(response=>{this.alertSaveResponse(response);this.getLoadStorageList()})
          break
        }
      }
    },
    getLoadProductList(){
      axios
          .get('http://95.216.8.101:8080/products/api/v1/loadProductList')
          .then(response=>{this.dbProductTable = response;
            this.tableReady = true})
    },
    getLoadStorageList(){
      axios
          .get('http://95.216.8.101:8080/storage/api/v1/loadStorageList')
          .then(response=>{this.dbStorageTable = response;this.showResponse()})
    }
  },
  computed:{
    rows(){
      return this.warehouse.length
    }
  },
  mounted(){
    this.getLoadProductList()
    this.getLoadStorageList()
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
