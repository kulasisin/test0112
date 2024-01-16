<template>
  <div class="login">
    <div class="container">
      <h1 class="my-3">Login page</h1>
    </div>
    <div class="container" v-if="!loginState">
      <v-form
        @submit="onSubmit"
        class="row g-3 needs-validation flex-column align-items-center"
        novalidate
        v-slot="{ errors }"
      >
        <div class="col-md-4 mb-3">
          <label for="username" class="form-label">Email address</label>
          <v-field
            type="email"
            name="Email Address"
            class="form-control"
            id="username"
            placeholder="name@example.com"
            :class="{ 'is-invalid': errors['Email Address'] }"
            rules="required|email"
            v-model="user.username"
          ></v-field>
          <error-message class="invalid-feedback" name="Email Address"></error-message>
        </div>
        <div class="col-md-4 mb-3">
          <label for="password" class="form-label">Password</label>
          <v-field
            type="password"
            name="Password"
            class="form-control"
            id="password"
            placeholder="請輸入密碼"
            :class="{ 'is-invalid': errors.Password }"
            rules="required"
            v-model="user.password"
          ></v-field>
          <error-message class="invalid-feedback" name="Password"></error-message>
        </div>
        <div class="col-md-4">
          <button
            class="btn w-100"
            :class="{
              'btn-secondary': defaultTheme === 'light',
              'btn-primary': defaultTheme === 'dark'
            }"
            type="submit"
          >
            Login
          </button>
        </div>
      </v-form>
      <div class="test"></div>
    </div>
  </div>
</template>
<script>
const apiUrl = import.meta.env.VITE_API
// vee-validate
import { Form, Field, ErrorMessage, defineRule, configure } from 'vee-validate'
import { required, email, regex } from '@vee-validate/rules'
defineRule('required', required)
defineRule('email', email)
defineRule('regex', regex)
import { localize, setLocale } from '@vee-validate/i18n'
// 下載語言
import zh_TW from '@vee-validate/i18n/dist/locale/zh_TW.json'

setLocale('zh_TW')
configure({
  // Generates an English message locale generator
  generateMessage: localize({ zh_TW }),
  validateOnInput: true // 當輸入任何內容直接進行驗證
})
export default {
  data() {
    return {
      user: {
        username: '',
        password: ''
      }
    }
  },
  components: {
    VForm: Form,
    VField: Field,
    ErrorMessage
  },
  props: ['loginState', 'defaultTheme'],
  methods: {
    onSubmit() {
      this.login()
    },
    async login() {
      try {
        this.$emit('emit-toggleLoading')
        const result = (await this.axios.post(`${apiUrl}v2/admin/signin`, this.user)).data
        const sweetConfig = {
          icon: 'success',
          title: result.message,
          timer: 1500
        }
        this.$swal(sweetConfig)
        const token = result.token
        document.cookie = `hexToken=${token};expires=${new Date(result.expired)}`
        this.user.username = ''
        this.user.password = ''
        this.$emit('emit-toggleLoading')
        await this.checkLogin()
        this.$router.push('products')
      } catch (err) {
        console.log(err)
        const sweetConfig = {
          icon: 'error',
          title: err.response.data.message
        }
        this.$swal(sweetConfig)
        this.$emit('emit-toggleLoading')
      }
    },
    async logOut() {
      try {
        const token = document.cookie.replace(
          /(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/,
          '$1'
        )
        this.axios.defaults.headers.common['Authorization'] = token
        const result = (await this.axios.post(`${apiUrl}v2/logout`)).data
        console.log(result)
      } catch (err) {
        console.log(err)
      }
    },
    async checkLogin() {
      try {
        this.$emit('emit-toggleLoading')
        const token = document.cookie.replace(
          /(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/,
          '$1'
        )
        this.axios.defaults.headers.common['Authorization'] = token
        const result = (await this.axios.post(`${apiUrl}v2/api/user/check`)).data
        if (result.success) {
          this.$emit('emit-toggleLogin', result.success)
        }
        this.$emit('emit-toggleLoading')
      } catch (err) {
        this.$emit('emit-toggleLogin', err.response.data.success)
        this.$emit('emit-toggleLoading')
      }
    }
  },
  created() {
    this.$emit('emit-login')
    this.checkLogin()
  },
  mounted() {}
}
</script>
<style lang="scss">
.test {
  background-image: url('/src/assets/images/components/test.jpg');
  height: 300px;
}
</style>
