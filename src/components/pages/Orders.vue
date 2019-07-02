<template>
    <div>
        <loading :active.sync="isLoading"></loading>

        <table class="table mt-4">
            <thead>
                <th width="120">購買時間</th>
                <th width="120">Email</th>
                <th>購買款項</th>
                <th width="120">應付金額</th>
                <th width="100">是否付款</th>
            </thead>
            <tbody>
                <tr v-for="(item, key) in orders" :key="key">
                    <td>{{item.create_at| date }}</td>
                    <td>{{item.user.email}}</td>
                    <td>
                        <ul class="list-unstyled">
                            <li v-for="(product, i) in item.products" :key="i">
                                {{ product.product.title }} 數量：{{ product.qty }} {{ product.product.unit }}
                            </li>
                        </ul>
                    </td>
                    <td class="text-right">{{item.total | currency}}</td>
                    <td>
                        <span v-if="item.is_paid == true" class="text-success">已付款</span>
                        <span v-else>未付款</span>
                    </td>
                </tr>
            </tbody>
        </table>

        <Pagination :pagination='pagination' @change-page='getOrders'/>
    </div>
</template>

<script>
import $ from 'jquery';
import Pagination from '../Pagination'

export default {
    name: 'Orders',
    components: {
        Pagination
    },
    data(){
        return{
            orders: [],
            pagination: {},
            isLoading: false
        }
    },
    created(){
        this.getOrders();
    },
    methods: {
        getOrders(page = 1){
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/orders?page=${page}`;
            vm.isLoading = true;
            this.$http.get(api).then((response) => {
                // console.log(response.data.orders);
                vm.orders = response.data.orders;
                vm.pagination = response.data.pagination;
                vm.isLoading = false;
            })
        },
        timeStamp2String (time){
            var datetime = new Date();
             datetime.setTime(time);
             var year = datetime.getFullYear();
             var month = datetime.getMonth() + 1;
             var date = datetime.getDate();
             var hour = datetime.getHours();
             var minute = datetime.getMinutes();
             var second = datetime.getSeconds();
             return year + "-" + month + "-" + date+" "+hour+":"+minute+":"+second;
        }
    }
}
</script>