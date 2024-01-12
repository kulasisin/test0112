<template>
  <div class="container">
    <h1>Home</h1>
  </div>
</template>
<script>
const apiUrl=import.meta.env.VITE_API;
const apiPath=import.meta.env.VITE_PATH;
export default {
  data(){
    return {

    }
  },
  methods: {
    async checkLogin(){
      try {
        this.$emit('emit-toggleLoading');
        const token=document.cookie.replace(/(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/,"$1",);
        this.axios.defaults.headers.common['Authorization'] = token;
        const result=(await this.axios.post(`${apiUrl}v2/api/user/check`)).data;
        if(result.success){
          this.$emit('emit-toggleLogin',result.success);
        }
        this.$emit('emit-toggleLoading');
      } catch (err) {
        this.$emit('emit-toggleLogin',err.response.data.success);
        this.$emit('emit-toggleLoading');
      }
    },
  },
  created(){
    this.$emit('emit-login');
    this.checkLogin();
  },
}
</script>