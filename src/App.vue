<template>
  <div id="loading" class="loadingBlock position-absolute z-1" v-if="loading">
    <img class="position-absolute top-50 start-50 translate-middle" src="./assets/images/components/loading2.svg" alt="loading pic">
  </div>
  <header>
    <nav class="navbar navbar-expand-lg" :data-bs-theme="theme">
      <div class="container position-relative">
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation" @click="toggleCollapse">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarSupportedContent" ref="collapse">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
              <li class="nav-item">
                <router-link class="nav-link" to="/">Home</router-link>
              </li>
              <li class="nav-item">
                <router-link class="nav-link" to="/products">Product</router-link>
              </li>
              <li class="nav-item" v-if="!loginState">
                <router-link class="nav-link" to="/login">Login</router-link>
              </li>
            </ul>
          </div>
          <div class="position-absolute top-0 end-0 me-3
position-absolute top-0 end-0 me-3 d-flex align-items-center">
            <div class="switchMode-box position-relative me-3">
              <img :src="themeIcon" alt="theme icon" class="switchBtn bg-light position-absolute" @click="changeTheme">
            </div>
            <button class="btn" :class="{'btn-secondary': theme==='light','btn-outline-primary': theme==='dark'}" v-if="loginState" @click="signOut">Sign Out</button>
            <router-link class="btn" :class="{'btn-secondary': theme==='light','btn-outline-primary': theme==='dark'}" to="/login" v-else-if="!loginState && !isLogin">Login</router-link>
          </div>
      </div>
    </nav>
  </header>
  <router-view @emit-login="isLoginPage" @emit-toggleLogin="toggleLoginState" @emit-toggleLoading="toggleLoading" :login-state="loginState" :default-theme="theme"></router-view>
</template>
<script>
const apiUrl=import.meta.env.VITE_API;
import sunIcon from '../src/assets/images/components/sun.png';
import moonIcon from '../src/assets/images/components/moon.png';
import Collapse from 'bootstrap/js/dist/collapse.js';
export default {
  data(){
    return {
      loginState: false,
      loading: false,
      isLogin: false,
      theme: 'light',
      themeIcon: sunIcon,
      bsCollapse: {},
    }
  },
  component :{
    Collapse,
  },
  provide(){
    return {
      loginState: this.loginState,
    }
  },
  watch: {
    theme(){
      if(this.themeIcon===sunIcon){
        this.themeIcon = moonIcon
      }else{
        this.themeIcon = sunIcon;
      }
    }
  },
  methods: {
    changeTheme(e){
      if(!e.target.className.includes('btnDark')){
        e.target.classList.add('btnDark');
        e.target.closest('.switchMode-box').classList.add('bg-primary');
      }else{
        e.target.classList.remove('btnDark');
        e.target.closest('.switchMode-box').classList.remove('bg-primary');
      }
      (() => {
        'use strict'

        const getStoredTheme = () => localStorage.getItem('theme')
        const setStoredTheme = theme => localStorage.setItem('theme', theme)
        
        const getPreferredTheme = () => {
          const storedTheme = getStoredTheme()
          if (storedTheme) {
            return storedTheme
          }
          if(this.theme==='light'){
            this.theme='dark';
            return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
          }else{
            this.theme='light';
            return window.matchMedia('(prefers-color-scheme: light)').matches ? 'dark' : 'light'
          }
        }

        const setTheme = theme => {
          if (theme === 'auto' && window.matchMedia('(prefers-color-scheme: dark)').matches) {
            document.documentElement.setAttribute('data-bs-theme', 'dark')
          } else {
            document.documentElement.setAttribute('data-bs-theme', theme)
          }
        }

        setTheme(getPreferredTheme())

        const showActiveTheme = (theme, focus = false) => {
          const themeSwitcher = document.querySelector('#bd-theme')

          if (!themeSwitcher) {
            return
          }

          const themeSwitcherText = document.querySelector('#bd-theme-text')
          const activeThemeIcon = document.querySelector('.theme-icon-active use')
          const btnToActive = document.querySelector(`[data-bs-theme-value="${theme}"]`)
          const svgOfActiveBtn = btnToActive.querySelector('svg use').getAttribute('href')

          document.querySelectorAll('[data-bs-theme-value]').forEach(element => {
            element.classList.remove('active')
            element.setAttribute('aria-pressed', 'false')
          })

          btnToActive.classList.add('active')
          btnToActive.setAttribute('aria-pressed', 'true')
          activeThemeIcon.setAttribute('href', svgOfActiveBtn)
          const themeSwitcherLabel = `${themeSwitcherText.textContent} (${btnToActive.dataset.bsThemeValue})`
          themeSwitcher.setAttribute('aria-label', themeSwitcherLabel)

          if (focus) {
            themeSwitcher.focus()
          }
        }

        window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', () => {
          const storedTheme = getStoredTheme()
          if (storedTheme !== 'light' && storedTheme !== 'dark') {
            setTheme(getPreferredTheme())
          }
        })

        window.addEventListener('DOMContentLoaded', () => {
          showActiveTheme(getPreferredTheme())

          document.querySelectorAll('[data-bs-theme-value]')
            .forEach(toggle => {
              toggle.addEventListener('click', () => {
                const theme = toggle.getAttribute('data-bs-theme-value')
                setStoredTheme(theme)
                setTheme(theme)
                showActiveTheme(theme, true)
              })
            })
        })
      })();
    },
    toggleLoading(){
      this.loading=!this.loading;
    },
    initBootstrapComponents(){
      const bsCollapse = new Collapse(this.$refs.collapse, {
        toggle: false
      });
      this.bsCollapse=bsCollapse;
    },
    toggleCollapse(){
      this.bsCollapse.toggle();
    },
    toggleLoginState(boolean){
      this.loginState=boolean;
    },
    isLoginPage(){
      if(location.href.split('/#/')[1]==='login'){
        this.isLogin=true;
      }else{
        this.isLogin=false;
      }
    },
    async signOut(){
      try {
        this.toggleLoading();
        const result=(await this.axios.post(`${apiUrl}v2/logout`)).data;
        this.toggleLoading();
        const sweetConfig={
          icon: 'success',
          title: result.message,
          timer: 1500
        }
        this.$swal(sweetConfig);
        document.cookie=`hexToken=;max-age=0;`;
        this.toggleLoginState();
        this.$router.push('/');
      } catch (err) {
        console.log(err);
        const sweetConfig={
          icon: 'error',
          title: err.response.data.message,
        }
        this.$swal(sweetConfig);
        this.toggleLoading();
      }
    },
  },
  created(){
    // this.changeTheme()
  },
  mounted(){
    this.initBootstrapComponents();
  }
}
</script>
<style lang="scss">
.loadingBlock{
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #0000005b;
}
.switchMode-box{
  transition: all 0.4s ease;
  width: 60px;
  height: 25px;
  background-color: #969393e6;
  border-radius: 100px;
  .switchBtn{
    transition: all 0.8s ease;
    background-size: cover;
    box-shadow: 0px 0px 5px 1px rgb(3, 3, 3);
    width: 25px;
    height: 25px;
    border-radius: 100px;
  }
  .btnDark{
    background-size: cover;
    margin-left: 35px;
    transition: all 0.8s ease;
  }
}
</style>