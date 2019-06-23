<template>
    <div>
        <loading :active.sync="isLoading"></loading>

        <table class="table mt-4">
            <thead>
                <th width="120">購買時間</th>
                <th>Email</th>
                <th width="120">購買款項</th>
                <th width="120">應付金額</th>
                <th width="100">是否付款</th>
                <th width="120">編輯</th>
            </thead>
            <!-- <tbody>
                <tr v-for="(item, key) in orders" :key="key">
                    <td>{{item.create_at}}</td>
                    <td>{{item.user.email}}</td>
                    <td>
                        <span v-for="n in item.products.length" :key='n'>
                            {{item.products[n].product_id}} 數量： {{item.products[n].qty}}
                        </span>
                    </td>
                    <td class="text-right">{{item.total | currency}}</td>
                    <td>
                        <span v-if="item.is_paid == true" class="text-success">已付款</span>
                        <span v-else>未付款</span>
                    </td>
                    <td>
                        <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
                        <button class="btn btn-outline-danger btn-sm" @click="openDeleteModal(item)">刪除</button>
                    </td>
                </tr>
            </tbody> -->
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
                console.log(response);
                vm.orders = response.data.orders;
                vm.pagination = response.data.pagination;
                vm.isLoading = false;
            })
        }
    }
}
</script>