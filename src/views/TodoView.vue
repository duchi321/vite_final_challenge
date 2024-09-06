<script setup>
import axios from 'axios'
import { useRouter } from 'vue-router'
import { ref, onMounted, computed } from 'vue'

const api = 'https://todolist-api.hexschool.io'
const router = useRouter()
const tokenCheck = ref('')
const signinStatus = ref({
  nickname: '',
  uid: ''
})
const newTodo = ref('')
const todes = ref([])
const showMessage = ref('')
const activeTag = ref('all')
const selectTag = (tag) => {
  activeTag.value = tag
}
// 未完成
const checkNot = computed(() => {
  return todes.value.filter((item) => item.status === false)
})

// 完成
const checkEnd = computed(() => {
  return todes.value.filter((item) => item.status === true)
})

// 驗證
onMounted(async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)cookiesSaveToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  tokenCheck.value = token
  if (!tokenCheck.value) {
    router.push('/login')
    return
  }
  try {
    const res = await axios.get(`${api}/users/checkout`, {
      headers: {
        Authorization: tokenCheck.value
      }
    })
    signinStatus.value = res.data
    getTodos()
  } catch (error) {
    console.log('未登入')
  }
})
// 登出
const signOut = async () => {
  if (signinStatus.value.uid === '') {
    alert('未登入')
    return
  }
  try {
    await axios.post(
      `${api}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: tokenCheck.value
        }
      }
    )
    alert('成功登出')
    router.push('/')
  } catch (error) {
    alert('登出錯誤: ' + error.message)
  }
}

// Todo List
const getTodos = async () => {
  try {
    const res = await axios.get(`${api}/todos/`, {
      headers: {
        Authorization: tokenCheck.value
      }
    })
    if (res.data.status) {
      if (res.data.data.length === 0) {
        showMessage.value = '目前尚無待辦事項'
        todes.value = res.data.data
      } else {
        showMessage.value = ''
        res.data.data.forEach((item) => {
          const createTime = item.createTime
          const date = new Date(createTime * 1000)
          const formattedDate = date.toISOString().slice(0, 19).replace('T', ' ')
          item.createTime = formattedDate
        })
        todes.value = res.data.data
      }
    }
  } catch (error) {
    console.log(error)
  }
}

// create
const createTodos = async () => {
  try {
    if (newTodo.value) {
      await axios.post(
        `${api}/todos/`,
        {
          content: newTodo.value
        },
        {
          headers: {
            Authorization: tokenCheck.value
          }
        }
      )
      newTodo.value = ''
      getTodos()
    } else {
      alert('沒有填寫代辦事項')
    }
  } catch (error) {
    console.log(error)
  }
}

const toggleTodos = async (id, status) => {
  try {
    await axios.patch(
      `${api}/todos/${id}/toggle`,
      {
        status: status
      },
      {
        headers: {
          Authorization: tokenCheck.value
        }
      }
    )
    getTodos()
  } catch (error) {
    console.log(error)
  }
}

const deleteTodos = async (id) => {
  try {
    await axios.delete(`${api}/todos/${id}`, {
      headers: {
        Authorization: tokenCheck.value
      }
    })
    getTodos()
  } catch (error) {
    console.log(error)
  }
}
</script>

<template>
  <!-- ToDo List -->
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm">
          <a
            ><span>Hello There {{ signinStatus.nickname }}!</span></a
          >
          <RouterLink to="" @click="signOut">登出</RouterLink>
        </li>
      </ul>
    </nav>
    <div class="conatiner todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="newTodo" />
          <a href="" @click.prevent="createTodos"> <i class="fa fa-plus"></i>+ </a>
        </div>
        <div class="todoList_list">
          <ul class="todoList_tab">
            <li>
              <a href="#" :class="{ active: activeTag === 'all' }" @click.prevent="selectTag('all')"
                >全部</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="{ active: activeTag === 'notyet' }"
                @click.prevent="selectTag('notyet')"
                >待完成</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="{ active: activeTag === 'theend' }"
                @click.prevent="selectTag('theend')"
                >已完成</a
              >
            </li>
          </ul>
          <div class="todoList_items" v-if="activeTag === 'all'">
            <ul class="todoList_item">
              <li v-for="item in todes" :key="item.id">
                <label class="todoList_label" :for="item.id">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :id="item.id"
                    v-model="item.status"
                    @change="toggleTodos(item.id)"
                  />
                  <span>{{ item.content }}</span>
                </label>
                <a href="" @click.prevent="deleteTodos(item.id)"><i class="delstyle">X</i></a>
              </li>
            </ul>
            <p>{{ showMessage }}</p>
            <div class="todoList_statistics" v-if="!showMessage">
              <p>{{ checkNot.length }} 個待完成項目</p>
            </div>
          </div>
          <div class="todoList_items" v-if="activeTag === 'notyet'">
            <ul class="todoList_item">
              <li v-for="item in checkNot" :key="item.id">
                <label class="todoList_label" :for="item.id">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :id="item.id"
                    v-model="item.status"
                    @change="toggleTodos(item.id)"
                  />
                  <span>{{ item.content }}</span>
                </label>
                <a href="" @click.prevent="deleteTodos(item.id)"><i class="delstyle">X</i></a>
              </li>
            </ul>
            <p>{{ showMessage }}</p>
            <div class="todoList_statistics" v-if="!showMessage">
              <p>{{ checkNot.length }} 個待完成項目</p>
            </div>
          </div>
          <div class="todoList_items" v-if="activeTag === 'theend'">
            <ul class="todoList_item">
              <li v-for="item in checkEnd" :key="item.id">
                <label class="todoList_label" :for="item.id">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :id="item.id"
                    v-model="item.status"
                    @change="toggleTodos(item.id)"
                  />
                  <span>{{ item.content }}</span>
                </label>
                <a href="" @click.prevent="deleteTodos(item.id)"><i class="delstyle">X</i></a>
              </li>
            </ul>
            <p>{{ showMessage }}</p>
            <div class="todoList_statistics" v-if="!showMessage">
              <p>{{ checkEnd.length }} 個已完成項目</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
