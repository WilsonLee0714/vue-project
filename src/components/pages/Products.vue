<template>
  <div>
    <Loading :active.sync="isLoading"></Loading>

    <div class="text-right mt-4">
      <button class="btn btn-primary" @click="openModal(true)">建立新的產品</button>
    </div>

    <table class="table mt-4">
      <thead>
        <tr>
          <th width="120">分類</th>

          <th>產品名稱</th>

          <th width="120">原價</th>

          <th width="120">售價</th>

          <th width="120">是否啟用</th>

          <th width="120">編輯</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="(item) in products" :key="item.id">
          <td>{{ item.category }}</td>

          <td>{{ item.title }}</td>

          <td class="text-right">{{ item.origin_price | currency }}</td>

          <td class="text-right">{{ item.price | currency }}</td>

          <td>
            <span v-if="item.is_enabled" class="text-success"></span>
            
            <span v-else>未啟用</span>
          </td>

          <td>
            <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
            
            <button class="btn btn-outline-danger btn-sm" @click="delproduct(item)">刪除</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Pagination -->
    <Pagination :pagination="pagination" @getProducts="getProducts"/>

    <!-- Modal -->
    <div
      class="modal fade"
      id="productModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-dark text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>新增產品</span>
            </h5>

            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>

          <div class="modal-body">
            <div class="row">
              <div class="col-sm-4">
                <div class="form-group">
                  <label for="image">輸入圖片網址</label>
                  
                  <input
                    type="text"
                    class="form-control"
                    id="image"
                    v-model="tempProduct.imageUrl"
                    placeholder="請輸入圖片連結"
                  >
                </div>

                <div class="form-group">
                  <label for="customFile">
                    或 上傳圖片
                    <i class="fas fa-spinner fa-spin" v-if="status.fileUploading"></i>
                  </label>
                  
                  <input
                    type="file"
                    id="customFile"
                    class="form-control"
                    ref="files"
                    @change="uploadFile"
                  >
                </div>

                <img
                  img="https://images.unsplash.com/photo-1483985988355-763728e1935b?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=828346ed697837ce808cae68d3ddc3cf&auto=format&fit=crop&w=1350&q=80"
                  class="img-fluid"
                  :src="tempProduct.imageUrl"
                  alt
                >
              </div>

              <div class="col-sm-8">
                <div class="form-group">
                  <label for="title">標題</label>
                  
                  <input
                    type="text"
                    class="form-control"
                    id="title"
                    v-model="tempProduct.title"
                    placeholder="請輸入標題"
                  >
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="category">分類</label>
                    
                    <input
                      type="text"
                      class="form-control"
                      id="category"
                      v-model="tempProduct.category"
                      placeholder="請輸入分類"
                    >
                  </div>

                  <div class="form-group col-md-6">
                    <label for="price">單位</label>
                    
                    <input
                      type="unit"
                      class="form-control"
                      id="unit"
                      v-model="tempProduct.unit"
                      placeholder="請輸入單位"
                    >
                  </div>
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="origin_price">原價</label>
                    
                    <input
                      type="number"
                      class="form-control"
                      id="origin_price"
                      v-model.number="tempProduct.origin_price"
                      placeholder="請輸入原價"
                    >
                  </div>

                  <div class="form-group col-md-6">
                    <label for="price">售價</label>
                    
                    <input
                      type="number"
                      class="form-control"
                      id="price"
                      v-model.number="tempProduct.price"
                      placeholder="請輸入售價"
                    >
                  </div>
                </div>

                <hr>

                <div class="form-group">
                  <label for="description">產品描述</label>
                  
                  <textarea
                    type="text"
                    class="form-control"
                    id="description"
                    v-model="tempProduct.description"
                    placeholder="請輸入產品描述"
                  ></textarea>
                </div>

                <div class="form-group">
                  <label for="content">說明內容</label>
                  
                  <textarea
                    type="text"
                    class="form-control"
                    id="content"
                    v-model="tempProduct.content"
                    placeholder="請輸入產品說明內容"
                  ></textarea>
                </div>

                <div class="form-group">
                  <div class="form-check">
                    <input
                      class="form-check-input"
                      type="checkbox"
                      v-model="tempProduct.is_enabled"
                      :true-value="1"
                      :false-value="0"
                      id="is_enabled"
                    >
                    
                    <label class="form-check-label" for="is_enabled">是否啟用</label>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
            
            <button type="button" class="btn btn-primary" @click="updateProduct">確認</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from "jquery";
import Pagination from "@/components/Pagination";

export default {
  data() {
    return {
      products: [],
      pagination: {},
      tempProduct: {},
      isNew: false,
      isLoading: false,
      status: {
        fileUploading: false
      }
    };
  },
  methods: {
    getProducts(page = 1) {
      const api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/products?page=${page}`;
      // console.log(process.env.APIPATH, process.env.CUSTOMPATH);
      this.isLoading = true;
      this.$http.get(api).then(res => {
        // console.log(res.data);
        this.isLoading = false;
        this.products = res.data.products;
        this.pagination = res.data.pagination;
      });
    },
    delproduct(item) {
      const api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/product/${item.id}`;
      // console.log(process.env.APIPATH, process.env.CUSTOMPATH);
      this.$http.delete(api).then(res => {
        // console.log(res.data);
        if (res.data.success) {
          this.getProducts();
        } else {
          this.getProducts();
          console.log("刪除失敗");
        }
      });
    },

    openModal(isNew, item) {
      if (isNew) {
        this.tempProduct = {};
        this.isNew = true;
      } else {
        this.tempProduct = Object.assign({}, item);
        this.isNew = false;
      }
      $("#productModal").modal("show");
    },
    updateProduct() {
      let api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/product`;
      let httpMethod = "post";
      if (!this.isNew) {
        api = `${process.env.APIPATH}/api/${
          process.env.CUSTOMPATH
        }/admin/product/${this.tempProduct.id}`;
        httpMethod = "put";
      }
      console.log(process.env.APIPATH, process.env.CUSTOMPATH);

      this.$http[httpMethod](api, {
        data: this.tempProduct
      }).then(res => {
        console.log(res.data);
        if (res.data.success) {
          $("#productModal").modal("hide");
          this.getProducts();
          this.tempProduct = {};
        } else {
          $("#productModal").modal("hide");
          this.getProducts();
          this.tempProduct = {};
          console.log("新增失敗");
        }
      });
    },
    uploadFile() {
      console.log(this);
      const uploadFile = this.$refs.files.files[0];
      const formData = new FormData();
      formData.append("file-to-upload", uploadFile);
      const url = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/upload`;
      this.status.fileUploading = true;
      this.$http
        .post(url, formData, {
          headers: {
            "Content-Type": "multipart/form-data"
          }
        })
        .then(res => {
          console.log(res.data);
          this.status.fileUploading = false;
          if (res.data.success) {
            this.$set(this.tempProduct, "imageUrl", res.data.imageUrl);
          } else {
            this.$bus.$emit("message:push", res.data.message, "danger");
          }
        });
    }
  },
  created() {
    this.getProducts();
  },
  components: {
    Pagination
  },
};
</script>