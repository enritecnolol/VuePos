<template>
  <div class="row" style="margin-top: 20px">
    <div class="col-md-12">
      <div class="card">
        <div class="card-header">
          <h3 class="card-title text-center">Productos</h3>
        </div>

        <div class="card-body">
          <div class="row">
            <div class="col-md-12">
              <div class="pull-right">
                <button type="button"
                        class="btn"
                        :class="showForm ? 'btn-danger':'btn-primary'"
                        style="float: right"
                        @click="showForm = !showForm">
                  <i :class="showForm ? 'icon-chevron-down' :'icon-plus22'"></i>
                  {{ showForm ? 'Ocultar':'Nuevo producto' }}
                </button>
                <button
                        type="button"
                        class="btn btn-primary"
                        data-toggle="modal"
                        data-target="#modal"
                        style="float: right; margin-right: 10px"><i class="icon-list"></i> Categorias</button>
              </div>
            </div>
            <div class="col-md-12" style="margin-top: 20px">
              <ProductForm
                      ref="ProductForm"
                      v-show="showForm"
                      :loading="loading"
                      @insert="insertProduct"
                      @edit="editProduct"
              />
            </div>
            <div class="col-md-12" style="margin: 10px" v-show="!showForm">
<!--              <select class="form-control" style="width: 100px" v-model="size_paginate">-->
<!--                <option value="5">5</option>-->
<!--                <option value="10">10</option>-->
<!--                <option value="25">25</option>-->
<!--              </select>-->
              <standardTable
                      v-if="!showForm"
                      :header="header"
                      :loading="loading"
                      :data="table_data"
                      :edit-body="true"
                      :pagination="pagination"
                      style="margin-top: 20px"
                      @changePage="changePage"
                      :search="true"
                      @size_paginateChange="size_paginateChange"
                      @searchItem="searchItem"
              >
                <template v-slot:extra-th>
                  <th></th>
                </template>
                <template v-slot:customization-body="{data}">
                  <td>{{data.id}}</td>
                  <td><img :src="data.img" width="60" alt=""></td>
                  <td>{{data.name}}</td>
                  <td>${{parseFloat(data.price).formatMoney(2)}}</td>
                  <td>{{data.barcode}}</td>
                  <td>{{CategoryName(data.categoria_id)}}</td>
                </template>
                <template v-slot:extra-td="{data}">
                  <td>
                    <Button class="btn btn-warning" @click="EditProduct(data)"><i class="icon-pencil"></i> Editar</Button>
                    <Button class="btn btn-danger" style="margin-left: 10px" @click="deleteProduct(data)"><i class="icon-trash"></i> Eliminar</Button>
                  </td>
                </template>
              </standardTable>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Categories></Categories>
  </div>
</template>
<script>
  import ProductForm from "../components/form/ProductForm";
  import Categories from "./Categories";
  import { ProductsServices } from "../service/ProductsService";
  import standardTable from "../components/table/standardTable";
  import { mapState } from 'vuex'

  export default {
    components:{
      ProductForm,
      Categories,
      standardTable
    },
    data(){
      return {
        size_paginate:5,
        loading:false,
        showForm:false,
        header:[],
        table_data:[],
        pagination:{}
      }
    },
    computed:{
      ...mapState({
        categories: state => state.categories.categories
      })
    },
    watch:{
      size_paginate()
      {
        this.getProductsPaginate()
      }
    },
    methods:{
      size_paginateChange(size)
      {
        this.size_paginate = size;
      },
      changePage({action, page})
      {
        if(action == 'next' && !page){
          this.getProductsPaginate(this.pagination.next_page_url)
        }else if (action == 'previous' && !page){
          this.getProductsPaginate(this.pagination.prev_page_url)
        }else{
          this.getProductsPaginate(this.pagination.path+'?page='+page)
        }
      },
      searchItem(search)
      {
        if(search == '')
        {
          this.getProductsPaginate();
          return
        }
        this.$store.dispatch('products/getsearchProduct', search)
                .then(res => {
                 this.table_data = res
                })
                .catch(error => {
                  console.log(error)
                })
      },
      getProductsPaginate(url)
      {
        this.loading = true;
        ProductsServices.getProductsPaginate({size:this.size_paginate, url:url})
                .then(res=> {
                  this.table_data =  res.data;
                  this.header = ['Id','','Nombre', 'Precio', 'Codigo de barra', 'Categoria'];
                  delete res['data'];
                  this.pagination = res;
                })
                .catch(err => {
                  this.$notify({
                    group: 'foo',
                    title: 'Productos',
                    text: err.response.data.message,
                    type: 'warn ',
                  });
                })
                .finally(()=> {
                  this.loading = false;
                })
      },
      EditProduct(data)
      {
        this.showForm = !this.showForm;
        this.$refs.ProductForm.ProductDataForEdit(data);
      },
      insertProduct(data)
      {
        this.loading = true;
        ProductsServices.insertProduct(data)
                .then(()=> {
                  this.$notify({
                    group: 'foo',
                    title: 'Operación Realizada',
                    text: 'El producto se ingreso con exito',
                    type: 'success ',
                  });
                  this.showForm = !this.showForm;
                  this.$refs.ProductForm.cleanForm();
                })
                .catch(err => {
                  this.$notify({
                    group: 'foo',
                    title: 'Productos',
                    text: err.response.data.message,
                    type: 'warn ',
                  });
                })
                .finally(()=> {
                  this.loading = false;
                })
      },
      CategoryName(CategoryId)
      {
        var found =  this.categories.find(category => category.id == CategoryId)
        if(found){
          return found.name
        }
      },
      editProduct(data)
      {
        this.loading = true;
        ProductsServices.editProduct(data)
                .then(()=> {
                  this.$notify({
                    group: 'foo',
                    title: 'Operación Realizada',
                    text: 'El producto se edito con exito',
                    type: 'success ',
                  });
                  this.getProductsPaginate();
                  this.$refs.ProductForm.cleanForm();
                })
                .catch(err => {
                  this.$notify({
                    group: 'foo',
                    title: 'Productos',
                    text: err.response.data.message,
                    type: 'warn ',
                  });
                })
                .finally(()=> {
                  this.loading = false;
                  this.showForm = !this.showForm;
                })
      },
      deleteProduct(data){
        this.loading = true;
        ProductsServices.deleteProduct(data)
                .then(() => {
                  this.$notify({
                    group: 'foo',
                    title: 'Operación Realizada',
                    text: 'El producto se elimino con exito',
                    type: 'success ',
                  });
                  this.getProductsPaginate();
                })
                .catch(err => {
                  this.$notify({
                    group: 'foo',
                    title: 'Productos',
                    text: err.response.data.message,
                    type: 'warn ',
                  });
                })
                .finally(()=> {
                  this.loading = false;
                })
      },
    },
    created() {
      this.getProductsPaginate();
      this.$store.dispatch('categories/getCategories');
    }
  }
</script>
