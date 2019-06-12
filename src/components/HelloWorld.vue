<template>
  <div class="hello">
    <a href="#" @click.prevent="signout">登出</a>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
    }
  },
  methods: {
    signout(){
      const api = `${process.env.APIPATH}/logout`;
      const vm = this;
      this.axios.post(api).then((response) => {
          console.log(response.data);
          if(response.data.success){
              vm.$router.push('/login')
          }
      })
    }
  }
}
</script>

<style scoped>

</style>
