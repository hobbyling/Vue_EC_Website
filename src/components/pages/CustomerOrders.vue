<template>
    <div>
        <loading :active.sync="isLoading"></loading>
        <div class="row mt-4">
            <div class="col-md-4 mb-4" v-for="(item, key) in products" :key="key">
                <div class="card border-0 shadow-sm">
                    <div style="height: 150px; background-size: cover; background-position: center" :style="{backgroundImage: `url(${item.imageUrl})`}"></div>
                    <div class="card-body">
                        <span class="badge badge-secondary float-right ml-2"> {{item.category}}</span>
                        <h5 class="card-title">
                            <a href="#" class="text-dark">{{item.title}}</a>
                        </h5>
                        <p class="card-text">{{item.contemt}}</p>
                        <div class="d-flex justify-content-between align-items-baseline">
                            <div class="h5" v-if="!item.price">{{item.origin_price}} 元</div>
                            <del class="h6" v-if="item.price">原價 {{item.origin_price}} 元</del>
                            <div class="h5" v-if="item.price">現在只要 {{item.price}} 元</div>
                        </div>
                    </div>
                    <div class="card-footer d-flex">
                        <button type="button" class="btn btn-outline-secondary btn-sm" @click="getProduct(item.id)">
                            <i class="fas fa-spinner fa-spin" v-if="status.loadingItem === item.id"></i>
                            查看更多
                        </button>
                        <button type="button" class="btn btn-outline-danger btn-sm ml-auto" @click="addtoCart(item.id)">
                            <i class="fas fa-spinner fa-spin" v-if="status.loadingItem === item.id"></i>
                            加到購物車
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div class="cartList container" v-if="carts">
            <table class="table">
                <thead>
                    <tr>
                        <th scope="col"></th>
                        <th scope="col">品名</th>
                        <th scope="col">數量</th>
                        <th scope="col">單價</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, key) in carts" :key="key">
                        <td>
                            <button class="btn btn-outline-danger">
                                <i class="fas fa-trash-alt"></i>
                            </button>
                        </td>
                        <td>
                            <span>{{item.product.title}}</span></br>
                            <span v-if="item.coupon" class="text-success">已套用優惠券</span>
                        </td>
                        <td>{{item.product.num}} / {{item.product.unit}}</td>
                        <td>{{item.product.price}}</td>
                    </tr>
                    <tr>
                        <td></td>
                        <td></td>
                        <td class="text-right">總計</td>
                        <td>{{total}}</td>
                    </tr>
                    <tr>
                        <td></td>
                        <td></td>
                        <td class="text-success text-right">折扣價</td>
                        <td class="text-success">{{finalTotal}}</td>
                    </tr>
                </tbody>
            </table>
            <div class="input-group mb-3">
                <input type="text" class="form-control" placeholder="請輸入優惠碼"  aria-describedby="button-addon2">
                <div class="input-group-append">
                    <button class="btn btn-outline-secondary" type="button" id="button-addon2">套用優惠碼</button>
                </div>
            </div>
        </div>

        <!-- Product Detail Modal -->
        <div class="modal fade" id="productModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLabel">
                            {{ product.title }}
                        </h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <img :src="product.imageUrl" alt="" class="img-fliud">

                        <blockquote class="blockquote mt-3">
                            <p class="mb-0">{{ product.contemt }}</p>
                            <footer class="blockquote-footer text-right">{{ product.description }}</footer>
                        </blockquote>

                        <div class="d-flex justify-content-between align-items-baseline">
                            <div class="h4" v-if="!product.price">{{product.origin_price}} 元</div>
                            <del class="h6" v-if="product.price">原價 {{product.origin_price}} 元</del>
                            <div class="h4" v-if="product.price">現在只要 {{product.price}} 元</div>
                        </div>

                        <select name="" id="" class="form-control mt-3" v-model="product.num">
                            <option :value="num" v-for="num in 10" :key="num">選購 {{num}} {{product.unit}}</option>
                        </select>
                    </div>
                    <div class="modal-footer">
                        <div class="text-muted text-nowrap mr-3">
                            小計 <strong> {{ product.num * product.price }} </strong> 元
                        </div>
                        <button type="button" class="btn btn-primary" @click="addtoCart(product.id, product.num)">
                            <i class="fas fa-spinner fa-spin" v-if="product.id === status.loadingItem"></i>
                            加到購物車
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';

export default {
    data(){
        return{
            products: [],
            product: {},
            carts: [],
            total: '',
            finalTotal: '',
            isLoading: false,
            status:{
                loadingItem: '',
            }
        }
    },
    created(){
        this.getProducts();
        this.getCart();
    },
    methods: {
        getProducts(){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/products/all`;
            vm.isLoading = true;
            this.$http.get(api).then((response) => {
                // console.log(response);
                vm.products = response.data.products;
                vm.isLoading = false;
            })
        },
        getProduct(id){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/product/${id}`;
            vm.status.loadingItem = id;
            this.$http.get(api).then((response) => {
                // console.log(response);
                $('#productModal').modal('show');
                vm.product = response.data.product;
                 vm.status.loadingItem = '';
                
            })
        },
        addtoCart(id, qty = 1){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/cart`;
            const cart = {
                product_id: id,
                qty
            }
            vm.status.loadingItem = id;
            this.$http.post(api, {data: cart}).then((response) => {
                console.log(response);
                vm.status.loadingItem = '';
                vm.getCart();
                $('#productModal').modal('hide');
            })
        },
        getCart(){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/cart`;
            vm.isLoading = true;
            this.$http.get(api).then((response) => {
                console.log(response);
                vm.carts = response.data.data.carts;
                vm.total = response.data.data.total;
                vm.finalTotal = response.data.data.final_total;
                vm.isLoading = false;
                
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.modal{
    img{
        width: 100%;
    }
}
</style>
