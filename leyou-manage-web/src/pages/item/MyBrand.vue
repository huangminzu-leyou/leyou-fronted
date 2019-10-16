<template>
  <div>
    <v-layout class="px-4 pb-3">
      <v-flex xs2>
        <v-btn class="info">新增品牌</v-btn>
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
    ></v-data-table>
  </div>
</template>

<script>
  export default {
    name: "MyBrand",

    data () {
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
      }
    },
    created () {
      this.loadBrands();
    },
    watch: {
      pagination: {
        key () {
          this.pagination.page = 1;
        },
        pagination: {
          deep: true,
          handler () {
            this.loadBrands();
          }
        }
      },
    },
    methods: {
      loadBrands () {
        debugger
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
