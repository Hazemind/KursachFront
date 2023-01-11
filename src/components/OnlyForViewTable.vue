<template>
  <div>
    <b-button variant="primary" v-on:click="showResponse">Показать записи</b-button>
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
          <b-form-input disabled v-model="row.item.cellNumber"/>
        </template>
        <template v-slot:cell(productNumber)="row">
          <b-form-input disabled v-model="row.item.productNumber"/>
        </template>
        <template v-slot:cell(slotSize)="row">
          <b-form-input disabled v-model="row.item.slotSize"/>
        </template>
        <template v-slot:cell(productName)="row">
          <b-form-select disabled v-model="row.item.productName" :options="productNamesList" />
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
        {key:'dateOfChange',label:'Дата изменения'}
      ],
      tableFilter: '',
      currentPage:1,
      perPage:11,
      filterOn:[],
      sortBy: 'cellNumber',
      tableReady:false
    }
  },
  props:[
    'login',
    'password'
  ],
  methods:{
    clearFilter(){
      this.tableFilter =''
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
