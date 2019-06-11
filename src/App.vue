<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App',
  created(){
    const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/products`;
    this.axios.get(api).then((response) => {
      console.log(response.data)
    })
  }
}
</script>

<style lang = 'scss'>
@import "./assets/all"
</style>
