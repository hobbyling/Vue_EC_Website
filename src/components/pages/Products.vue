<template>
    <div>
        <loading :active.sync="isLoading"></loading>
        <div class="text-right mt-4">
            <button class="btn btn-primary" @click="openModal(true)">建立新的產品</button>
        </div>

        <table class="table mt-4">
            <thead>
                <th width="120">分類</th>
                <th>產品名稱</th>
                <th width="120">原價</th>
                <th width="120">售價</th>
                <th width="100">是否啟用</th>
                <th width="120">動作</th>
            </thead>
            <tbody>
                <tr v-for="(item, key) in products" :key="key">
                    <td>{{item.category}}</td>
                    <td>{{item.title}}</td>
                    <td class="text-right">{{item.origin_price | currency}}</td>
                    <td class="text-right">{{item.price | currency}}</td>
                    <td>
                        <span v-if="item.is_enabled == 1" class="text-success">啟用</span>
                        <span v-else>未啟用</span>
                    </td>
                    <td>
                        <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
                        <button class="btn btn-outline-danger btn-sm" @click="openDeleteModal(item)">刪除</button>
                    </td>
                </tr>
            </tbody>
        </table>

        <Pagination :pagination='pagination' @change-page='getProducts'/>

        <!-- update Modal -->
        <div class="modal fade" id="productModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content border-0">
                    <div class="modal-header bg-dark text-white">
                        <h5 class="modal-title" id="exampleModalLabel">
                            <span v-if="isNew">新增產品</span>
                            <span v-else>編輯產品</span>
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
                                    <input type="text" class="form-control" id="image" placeholder="請輸入圖片連結" v-model="tmpProduct.imageUrl">
                                </div>
                                <div class="form-group">
                                    <label for="customFile">或 上傳圖片
                                        <i class="fas fa-spinner fa-spin" v-if="status.fileUploading"></i>
                                    </label>
                                    <input type="file" id="customFile" class="form-control" ref="files" @change="uploadFile()">
                                </div>
                                <img img="https://images.unsplash.com/photo-1483985988355-763728e1935b?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=828346ed697837ce808cae68d3ddc3cf&auto=format&fit=crop&w=1350&q=80" class="img-fluid" alt="" :src="tmpProduct.imageUrl">
                            </div>
                            <div class="col-sm-8">
                                <div class="form-group">
                                    <label for="title">標題</label>
                                    <input type="text" name="title" class="form-control" id="title" placeholder="請輸入標題" v-model="tmpProduct.title" v-validate="'required'">
                                    <span class="text-danger" v-if="errors.has('title')">{{errors.first('title') }}</span>
                                </div>

                                <div class="form-row">
                                    <div class="form-group col-md-6">
                                        <label for="category">分類</label>
                                        <input type="text" name="category" class="form-control" id="category" placeholder="請輸入分類" v-model="tmpProduct.category" v-validate="'required'">
                                        <span class="text-danger" v-if="errors.has('category')">{{errors.first('category') }}</span>
                                    </div>
                                    <div class="form-group col-md-6">
                                        <label for="price">單位</label>
                                        <input type="unit" name="unit" class="form-control" id="unit" placeholder="請輸入單位" v-model="tmpProduct.unit" v-validate="'required'">
                                        <span class="text-danger" v-if="errors.has('unit')">{{errors.first('unit') }}</span>
                                    </div>
                                </div>

                                <div class="form-row">
                                    <div class="form-group col-md-6">
                                        <label for="origin_price">原價</label>
                                        <input type="number" class="form-control" id="origin_price" placeholder="請輸入原價" v-model="tmpProduct.origin_price" v-validate="'required'">
                                        <span class="text-danger" v-if="errors.has('origin_price')">{{errors.first('origin_price') }}</span>
                                    </div>
                                    <div class="form-group col-md-6">
                                        <label for="price">售價</label>
                                        <input type="number" class="form-control" id="price" placeholder="請輸入售價" v-model="tmpProduct.price" v-validate="'required'">
                                        <span class="text-danger" v-if="errors.has('price')">{{errors.first('price') }}</span>
                                    </div>
                                </div>
                                <hr>

                                <div class="form-group">
                                    <label for="description">產品描述</label>
                                    <textarea type="text" class="form-control" id="description" placeholder="請輸入產品描述" v-model="tmpProduct.description"></textarea>
                                </div>
                                <div class="form-group">
                                    <label for="content">說明內容</label>
                                    <textarea type="text" class="form-control" id="content" placeholder="請輸入產品說明內容" v-model="tmpProduct.contemt"></textarea>
                                </div>
                                <div class="form-group">
                                    <div class="form-check">
                                        <input class="form-check-input" type="checkbox" id="is_enabled" v-model="tmpProduct.is_enabled" :true-value="1" :false-value="0">
                                        <label class="form-check-label" for="is_enabled">
                                            是否啟用
                                        </label>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
                        <button type="button" class="btn btn-primary" @click="updateProduct()">確認</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- delete Modal -->
        <div class="modal fade" id="delProductModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content border-0">
                    <div class="modal-header bg-danger text-white">
                        <h5 class="modal-title" id="exampleModalLabel">
                            <span>刪除產品</span>
                        </h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        是否刪除 <strong class="text-danger">{{ tmpProduct.title }}</strong> 商品(刪除後將無法恢復)。
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
                        <button type="button" class="btn btn-danger" @click="deleteProduct()">確認刪除</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';
import Pagination from '../Pagination'

export default {
    name: 'Products',
    components: {
        Pagination
    },
    data(){
        return{
            products: [],
            pagination: {},
            tmpProduct: {},
            isNew: false,
            isLoading: false,
            status: {
                fileUploading: false
            }
        }
    },
    methods: {
        getProducts(page = 1){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/products?page=${page}`;
            vm.isLoading = true;
            this.$http.get(api).then((response) => {
                console.log(response);
                vm.products = response.data.products;
                vm.pagination = response.data.pagination;
                vm.isLoading = false;
            })
        },
        updateProduct(){
            const vm = this;
            let api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product`;
            let httpMethod = 'post';
            if(!vm.isNew){
                api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product/${vm.tmpProduct.id}`;
                httpMethod = 'put';
            }
            this.$validator.validate().then((valid) => {
                if (valid) {
                    this.$http[httpMethod](api, {data: vm.tmpProduct}).then((response) => {
                        console.log(response.data)
                        if(response.data.success){
                            $('#productModal').modal('hide');
                            vm.getProducts();
                        }else{
                            $('#productModal').modal('hide');
                            vm.getProducts();
                            console.log('新增失敗');
                        }
                    })
                }else{
                    console.log('欄位不完整')
                }
            })
        },
        deleteProduct(){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product/${vm.tmpProduct.id}`;
            this.$http.delete(api, {data: vm.tmpProduct}).then((response) => {
                console.log(response.data);
                if(response.data.success){
                    $('#delProductModal').modal('hide');
                    vm.getProducts();
                }else{
                    $('#delProductModal').modal('hide');
                    vm.getProducts();
                    console.log('刪除失敗');
                }
            })
        },
        openModal(isNew, item){
            if(isNew){
                this.tmpProduct = {};
                this.isNew = true;
            }else{
                this.tmpProduct = Object.assign({}, item);
                this.isNew = false;
            }
            $('#productModal').modal('show');
        },
        openDeleteModal(item){
            this.tmpProduct = Object.assign({}, item);
            $('#delProductModal').modal('show');
        },
        uploadFile(){
            console.log(this);
            const uploadedFile = this.$refs.files.files[0];
            const vm = this;
            const formData = new FormData();
            formData.append('file-to-upload', uploadedFile);
            const url = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/upload`; 
            vm.status.fileUploading = true;
            this.$http.post(url, formData, {
                headers:{
                    'Content-Type': 'multipart/form-data'
                }
            }).then((response) => {
                console.log(response.data);
                if(response.data.success){
                    vm.$set(vm.tmpProduct, 'imageUrl', response.data.imageUrl)
                }else{
                    this.$bus.$emit('message:push', response.data.message, 'danger')
                }
                console.log(vm.tmpProduct)
                vm.status.fileUploading = false
            })
        }
    },
    created(){
        this.getProducts();
    }
    
}
</script>