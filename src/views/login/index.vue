<!--suppress ALL -->
<template>
  <div class="login-container">
    <div class="wrapper">
      <!--  Header  -->
      <header class="section header">
        <div class="trapezoid" />
        <div class="header__text">
          <h1>sup.</h1>
          <p>Sign in or create a new account.</p>
        </div>
      </header>
      <!-- Sign In -->
      <section class="section sign-in">
        <el-form ref="refLoginForm" class="form" :model="subForm" :rules="formRules">
          <el-form-item prop="keyword" :rules="formRules.isNotNull('usename不能为空')">
            <el-input v-model="subForm.keyword" placeholder="用户名(admin)" />
            <div class="show-pwd" />
          </el-form-item>
          <el-form-item prop="password" :rules="formRules.isNotNull('密码不能为空')">
            <el-input
              :key="passwordType"
              ref="refPassword"
              v-model="subForm.password"
              :type="passwordType"
              name="password"
              placeholder="password(123456)"
              @keyup.enter="handleLogin"
            />
            <!-- <span class="show-pwd" @click="showPwd">
              <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
            </span> -->
          </el-form-item>
          <div v-if="tipMessage" class="tip-message">{{ tipMessage }}</div>
          <el-button :loading="subLoading" type="primary" class="login-btn" size="default" @click.prevent="handleLogin">
            Login
          </el-button>
        </el-form>
      </section>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useBasicStore } from '@/store/basic'
import { elMessage, useElement } from '@/hooks/use-element'
import { loginReq } from '@/api/user'

/* listen router change and set the query  */
const { settings } = useBasicStore()
//element valid
const formRules = useElement().formRules
//form
const subForm = reactive({
  keyword: 'panda',
  password: '123456'
})
const state = reactive({
  otherQuery: {},
  redirect: undefined
})
const route = useRoute()
const getOtherQuery = (query) => {
  return Object.keys(query).reduce((acc, cur) => {
    if (cur !== 'redirect') {
      acc[cur] = query[cur]
    }
    return acc
  }, {})
}
watch(
  () => route.query,
  (query) => {
    if (query) {
      state.redirect = query.redirect
      state.otherQuery = getOtherQuery(query)
    }
  },
  { immediate: true }
)

/*
 *  login relative
 * */
let subLoading = $ref(false)
//tip message
let tipMessage = $ref('')
//sub form
const refLoginForm = $ref(null)
const handleLogin = () => {
  refLoginForm.validate((valid) => {
    subLoading = true
    if (valid) loginFunc()
  })
}
const router = useRouter()
const basicStore = useBasicStore()

const loginFunc = () => {
  loginReq(subForm)
    .then(({ data }) => {
      elMessage('登录成功')
      basicStore.setToken(data?.jwtToken)
      router.push('/')
    })
    .catch((err) => {
      tipMessage = err?.msg
    })
    .finally(() => {
      subLoading = false
    })
}
/*
 *  password show or hidden
 * */
const passwordType = ref('password')
const refPassword = ref(null)
const showPwd = () => {
  if (passwordType.value === 'password') {
    passwordType.value = ''
  } else {
    passwordType.value = 'password'
  }
  nextTick(() => {
    refPassword.value.focus()
  })
}
</script>

<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;
// Colors
$c-blue: #1126be;
$c-white: #fefefe;

// Transparent Colors
$t-blue: rgba(17, 38, 190, 0.8);

// Gradients
$g-blue: linear-gradient($t-blue, $t-blue);

// Shadows
$s-basic: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
.login-container {
  height: 100vh;
  width: 100%;
  background-color: #fefefe;

  form {
    margin: 0 auto;
    max-width: 17rem;
    overflow: auto;
    > button {
      margin-top: 2rem;
      border: 0;
      border-radius: 200px;
      width: 100%;
      padding: 0.85rem;
      font-size: 1rem;
      color: #fefefe;
      background: #1126be;

      &:focus {
        outline: none;
      }
    }
  }

  .wrapper {
    margin: 0 auto;
    width: 400px;
    box-shadow: $s-basic;
  }

  .section {
    padding: 1rem;
  }

  // Header Styles
  .header {
    position: relative;
    text-align: center;

    &__text {
      position: relative;
      padding: 3.5rem 0 7rem;

      > h1 {
        margin: 0;
        font-size: 2.5rem;
      }
    }

    > .trapezoid {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: skewY(-10deg);
      transform-origin: top left;
      box-shadow: $s-basic;
      background: $g-blue,
        url('https://images.pexels.com/photos/520936/pexels-photo-520936.jpeg?w=940&h=650&auto=compress&cs=tinysrgb');
      background-position: top center;
      background-attachment: fixed;
    }
  }

  .title-container {
    .title {
      font-size: 22px;
      color: #eee;
      margin: 0px auto 25px auto;
      text-align: center;
      font-weight: bold;
    }
  }
}

.svg-container {
  padding-left: 6px;
  color: $dark_gray;
  text-align: center;
  width: 30px;
}

//错误提示信息
.tip-message {
  color: #e4393c;
  height: 30px;
  margin-top: -12px;
  font-size: 12px;
}

//登录按钮
.login-btn {
  width: 100%;
  margin-bottom: 30px;
}
.show-pwd {
  width: 50px;
  font-size: 16px;
  color: $dark_gray;
  cursor: pointer;
  text-align: center;
}
</style>

<style lang="scss">
//css 样式重置 增加个前缀避免全局污染
.login-container {
  .el-input__wrapper {
    background-color: transparent;
    box-shadow: none;
  }
  .el-input {
    background: none;
  }
  .el-input input {
    border: 0;
    border-bottom: 1px solid #1126be;
    border-radius: 0;
    width: 100%;
    height: 2rem;
    padding: 0 0 0.25rem 0;
    font-size: 1rem;
    color: #1126be;
    background: #fefefe;

    &:focus {
      outline: none;
    }

    &::placeholder {
      color: #1126be;
    }
  }

  //hiden the input border
  .el-input__inner {
    box-shadow: none !important;
  }
}
</style>
