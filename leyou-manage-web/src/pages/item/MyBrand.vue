<template>
  <div>
    <v-layout class="px-4 pb-3">
      <v-flex xs2>
        <v-btn @click="addBrand" color="primary">新增品牌</v-btn>
      </v-flex>
      <v-spacer/>
      <v-flex xs6>
        <v-text-field label="搜索" hide-details append-icon="search" v-model="key"></v-text-field>
      </v-flex>
    </v-layout>
    <v-data-table
      :headers="headers"
      :items="brands"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img v-if="props.item.image" :src="props.item.image" width="130" height="40"/></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td>
          <v-btn flat icon color="info">
            <v-icon>edit</v-icon>
          </v-btn>
          <v-btn flat icon color="error">
            <v-icon>delete</v-icon>
          </v-btn>
        </td>
      </template>
    </v-data-table>

    <v-dialog v-model="show" max-width="600" scrollable v-if="show">
      <v-card>
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="show = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5 py-2">
          <!-- 表单 -->
          <brand-form :oldBrand="brand" :isEdit="isEdit" @close="show = false" :reload="getDataFromApi"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  import BrandForm from './BrandForm'
  import {brandData} from '../../mockDB'

  export default {
    comments: {
      BrandForm
    },

    name: "MyBrand",

    data() {
      return {
        headers: [// 表头
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false}
        ],
        brands: [],
        pagination: {},// 分页信息
        totalBrands: 0,
        loading: true,
        key: '',
        show: false,// 是否弹出窗口
        isEdit: false // 判断是编辑还是新增
      }
    },
    created() {
      this.loadBrands();
    },
    watch: {
      key() {
        this.pagination.page = 1;
        this.loadBrands();
      },
      pagination: {
        handler() {
          this.loadBrands();
        },
        deep: true
      }
    },
    methods: {
      addBrand() {
        this.brand = {};
        this.isEdit = false;
        this.show = true;
      },
      editBrand(item) {
        this.brand = item;
        this.isEdit = true;
        this.show = true;
        // 查询商品分类信息，进行回显
        this.$http.get("/item/category/bid/" + item.id)
          .then(resp => {
            this.brand.categories = resp.data;
          })

      },
      deleteBrand(item) {
        this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(() => {
          // 发起删除请求
          this.$http.delete("/item/brand?id=" + item.id,)
            .then(() => {
              // 删除成功，重新加载数据
              this.$message.success("删除成功！");
              this.getDataFromApi();
            })
        }).catch(() => {
          this.$message.info("删除已取消！");
        });

      },
      loadBrands() {
        this.loading = true;
        // 200ms后返回假数据
        this.$http.get('/item/brand/page', {
          params: {
            page: this.pagination.page,//当前页
            rows: this.pagination.rowsPerPage,//每页大小
            sortBy: this.pagination.sortBy, //分页字段
            desc: this.pagination.descending,//第几页
            key: this.key,//查询条件
          }
        })
          .then(resp => {
            this.brands = resp.data.items;
            this.totalBrands = resp.data.total;
          })
      }
    }
  }
</script>
