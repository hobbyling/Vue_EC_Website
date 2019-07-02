<template>
    <div>
        <loading :active.sync="isLoading"></loading>
        <div class="text-right mt-4">
            <button class="btn btn-primary" @click="openModal(true)">建立新的優惠券</button>
        </div>

        <table class="table mt-4">
            <thead>
                <th>名稱</th>
                <th width="80">折扣百分比</th>
                <th width="100">是否啟用</th>
                <th width="150">到期日</th>
                <th width="150">動作</th>
            </thead>
            <tbody>
                <tr v-for="(item, key) in coupons" :key="key">
                    <td>{{item.title}}</td>
                    <td>{{item.percent}}</td>
                    <td>
                        <span v-if="item.is_enabled == 1" class="text-success">啟用</span>
                        <span v-else>未啟用</span>
                    </td>
                    <td>{{item.due_date}}</td>
                    <td>
                        <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
                        <button class="btn btn-outline-danger btn-sm" @click="openDeleteModal(item)">刪除</button>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- update Modal -->
        <div class="modal fade" id="couponModal" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content border-0">
                    <div class="modal-header bg-dark text-white">
                        <h5 class="modal-title" id="exampleModalLabel">
                            <span v-if="isNew">新增優惠券</span>
                            <span v-else>編輯優惠券</span>
                        </h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-sm-12">
                                <div class="form-group">
                                    <label for="title">標題</label>
                                    <input type="text" class="form-control" id="title" placeholder="請輸入標題" v-model="tmpCoupon.title">
                                </div>
                                <div class="form-row">
                                    <div class="form-group col-md-6">
                                        <label for="category">折數</label>
                                        <input type="number" class="form-control" id="category" placeholder="請輸入折數" v-model="tmpCoupon.percent">
                                    </div>
                                    <div class="form-group col-md-6">
                                        <label for="price">到期日</label>
                                        <input type="date" class="form-control" id="unit" placeholder="請輸入到期日" v-model="tmpCoupon.due_date">
                                    </div>
                                </div>
                                <div class="form-group">
                                    <div class="form-check">
                                        <input class="form-check-input" type="checkbox" id="is_enabled" v-model="tmpCoupon.is_enabled" :true-value="1" :false-value="0">
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
                        <button type="button" class="btn btn-primary" @click="updateCoupon()">確認</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- delete Modal -->
        <div class="modal fade" id="delCouponModal" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content border-0">
                    <div class="modal-header bg-danger text-white">
                        <h5 class="modal-title" id="exampleModalLabel">
                            <span>刪除優惠券</span>
                        </h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        是否刪除 <strong class="text-danger">{{ tmpCoupon.title }}</strong> 優惠券(刪除後將無法恢復)。
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
                        <button type="button" class="btn btn-danger" @click="deleteCoupon()">確認刪除</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';

export default {
    name: 'Coupons',
    data(){
        return{
            isLoading: false,
            coupons: [],
            pagination: {},
            tmpCoupon: {},
            isNew: false
        }
    },
    created(){
        this.getCoupons();
    },
    methods: {
        getCoupons(page = 1){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupons?page=${page}`;
            vm.isLoading = true;
            this.$http.get(api).then((response) => {
                console.log(response);
                vm.coupons = response.data.coupons;
                vm.pagination = response.data.pagination;
                vm.isLoading = false;
            })
        },
        updateCoupon(){
            const vm = this;
            let api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon`;
            let httpMethod = 'post';
            if(!vm.isNew){
                api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon/${vm.tmpCoupon.id}`;
                httpMethod = 'put';
            }
            this.$http[httpMethod](api, {data: vm.tmpCoupon}).then((response) => {
                console.log(response.data)
                if(response.data.success){
                    $('#couponModal').modal('hide');
                    vm.getCoupons();
                }else{
                    $('#couponModal').modal('hide');
                    vm.getCoupons();
                    console.log('新增失敗');
                }
            })
        },
        deleteCoupon(){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon/${vm.tmpCoupon.id}`;
            this.$http.delete(api, {data: vm.tmpCoupon}).then((response) => {
                console.log(response.data);
                if(response.data.success){
                    $('#delCouponModal').modal('hide');
                    vm.getCoupons();
                }else{
                    $('#delCouponModal').modal('hide');
                    vm.getCoupons();
                    console.log('刪除失敗');
                }
            })
        },
        openModal(isNew, item){
            if(isNew){
                this.tmpCoupon = {};
                this.isNew = true;
            }else{
                this.tmpCoupon = Object.assign({}, item);
                this.isNew = false;
            }
            $('#couponModal').modal('show');
        },
        openDeleteModal(item){
            this.tmpCoupon = Object.assign({}, item);
            $('#delCouponModal').modal('show');
        },
    }
}
</script>