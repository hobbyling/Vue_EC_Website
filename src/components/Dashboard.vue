<template>
    <div>
        <Sidebar/>
        <div class="container-fluid">
            <div class="row">
                <Navbar/>

                <main role="main" class="col-md-9 ml-sm-auto col-lg-10 px-4">
                    <router-view></router-view>
                </main>
            </div>
        </div>
    </div>
</template>

<script>
import Sidebar from './Sidebar';
import Navbar from './Navbar'

export default {
    components:{
        Sidebar,
        Navbar
    }
}
</script>