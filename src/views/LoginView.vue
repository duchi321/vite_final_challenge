<script setup>
import axios from 'axios'
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const api = 'https://todolist-api.hexschool.io'
const tokenCheck = ref('')
const router = useRouter()

// 登入
const signinData = ref({
  email: '',
  password: ''
})

const signinStatus = ref({
  nickname: '',
  uid: ''
})
const signIn = async () => {
  if (signinStatus.value.uid !== '') {
    alert('已登入')
    return
  }
  try {
    const res = await axios.post(`${api}/users/sign_in`, signinData.value)
    signinStatus.value = res.data.status
    document.cookie = `cookiesSaveToken=${res.data.token}; expires=${res.data.exp}; path=/`
    location.reload()
    alert('登入')
  } catch (err) {
    alert('帳號或密碼錯誤!')
  }
}

//驗證----------------------

onMounted(async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)cookiesSaveToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  tokenCheck.value = token
  document.cookie = `cookiesSaveToken = ${tokenCheck.value}`
  try {
    const res = await axios.get(`${api}/users/checkout`, {
      headers: {
        Authorization: tokenCheck.value
      }
    })
    signinStatus.value = res.data
    router.push({ path: '/todolist' })
  } catch (error) {
    console.log('未登入')
  }
})
</script>
<template>
  <!-- login_page -->
  <div id="loginPage" class="bg-yellow">
    <div class="conatiner loginPage vhContainer">
      <div class="side">
        <a href="#"
          ><img
            class="logoImg"
            src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
            alt=""
        /></a>
        <img
          class="d-m-n"
          src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
          alt="workImg"
        />
      </div>
      <div>
        <form class="formControls" action="index.html">
          <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
          <label class="formControls_label" for="email">Email</label>
          <input
            class="formControls_input"
            type="text"
            id="email"
            name="email"
            placeholder="請輸入 email"
            v-model="signinData.email"
            required
          />
          <span v-if="!signinData.email">此欄位不可留空</span>
          <label class="formControls_label" for="pwd">密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            placeholder="請輸入密碼"
            v-model="signinData.password"
            required
          />
          <span v-if="!signinData.password">此欄位不可留空</span>
          <input class="formControls_btnSubmit" type="button" @click="signIn" value="登入" />
          <RouterLink class="formControls_btnLink" to="/signup">註冊帳號</RouterLink>
        </form>
      </div>
    </div>
  </div>
</template>
